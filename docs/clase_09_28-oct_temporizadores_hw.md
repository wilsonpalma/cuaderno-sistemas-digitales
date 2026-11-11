# Clase 9: Temporizadores de hardware en el ESP32

**Asignatura:** Sistemas Digitales  
**Fecha:** Miércoles 28 de octubre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche  
**Estudiante:** Wilson Patricio Palma Samaniego

## Objetivo de la clase

Conocer cómo funcionan los temporizadores de hardware del ESP32, identificar sus componentes principales y configurar un temporizador mediante un prescalador, una alarma y la opción de reinicio automático (*auto-reload*).

## ¿Qué es un temporizador de hardware?

Un temporizador de hardware es un periférico que cuenta pulsos de reloj sin que la CPU tenga que incrementar el contador mediante instrucciones. Su funcionamiento general es:

```text
Reloj APB (80 MHz)
        |
        v
   Prescalador
        |
        v
 Contador de 64 bits
        |
        v
 Comparador / Alarma
        |
        v
    Interrupción
```

El conteo es independiente del programa principal. Al alcanzar el valor configurado, el temporizador genera una interrupción que puede utilizarse para leer sensores o controlar señales periódicas.

## Arquitectura de los temporizadores del ESP32

Según lo revisado en clase, el ESP32 dispone de dos grupos de temporizadores (*Timer Groups*). Cada grupo contiene dos temporizadores de propósito general, por lo que se cuenta con cuatro temporizadores independientes de 64 bits:

```text
ESP32
├── Timer Group 0
│   ├── Timer 0 (64 bits)
│   └── Timer 1 (64 bits)
└── Timer Group 1
    ├── Timer 2 (64 bits)
    └── Timer 3 (64 bits)
```

El prescalador permite ajustar la resolución del conteo.

## Reloj base y prescalador

La fuente de reloj de referencia para estos periféricos es el reloj APB, que en el material de clase se considera de 80 MHz. Como esta frecuencia es elevada para medir intervalos de uso común, se utiliza un prescalador para dividirla.

La relación es:

\[
f_{\text{contador}} = \frac{f_{\text{APB}}}{\text{prescalador}}
\]

Por ejemplo, al configurar un prescalador de 80:

\[
\frac{80\,000\,000}{80}=1\,000\,000\text{ Hz}
\]

El contador trabaja entonces a 1 MHz. Esto significa que cada incremento, o *tick*, representa un microsegundo. De esta forma, resulta más sencillo calcular el valor que debe utilizarse en la alarma.

## Alarma y auto-reload

La alarma define cuántos ticks debe contar el temporizador antes de generar un evento y ejecutar la ISR asociada. Con *auto-reload* activado, el contador vuelve a cero automáticamente y comienza un nuevo período.

## Funciones principales de la API de Arduino para ESP32

En la sesión se revisaron las siguientes funciones:

| Función | Propósito |
|---|---|
| `timerBegin(uint8_t timer, uint16_t divider, bool countUp)` | Inicializa el temporizador, establece el prescalador y el sentido del conteo. |
| `timerAttachInterrupt(hw_timer_t *timer, void (*fn)(void), bool edge)` | Asocia una función ISR al temporizador. |
| `timerAlarmWrite(hw_timer_t *timer, uint64_t interruptAt, bool autoreload)` | Define el valor de alarma y activa o desactiva el auto-reload. |
| `timerAlarmEnable(hw_timer_t *timer)` | Habilita la alarma del temporizador. |

## Ejemplo: parpadeo de LED con una alarma periódica

El siguiente programa configura el Timer 0 con un prescalador de 80. Como cada tick representa un microsegundo, una alarma de 1 000 000 ticks corresponde a un segundo.

```cpp
#include <Arduino.h>

const uint8_t LED_PIN = 2;
hw_timer_t *miTimer = NULL;

volatile bool estadoLED = false;
volatile uint32_t contadorInterrupciones = 0;

void IRAM_ATTR alDispararAlarma() {
  estadoLED = !estadoLED;
  contadorInterrupciones++;
}

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);

  // Timer 0, prescalador 80 y conteo ascendente
  miTimer = timerBegin(0, 80, true);

  // Asociar la rutina de interrupción
  timerAttachInterrupt(miTimer, &alDispararAlarma, true);

  // Alarma de un segundo con reinicio automático
  timerAlarmWrite(miTimer, 1000000, true);

  // Habilitar la alarma
  timerAlarmEnable(miTimer);

  Serial.println("Temporizador configurado a 1 Hz.");
}

void loop() {
  digitalWrite(LED_PIN, estadoLED);

  static uint32_t ultimoReporte = 0;
  if (millis() - ultimoReporte >= 3000) {
    ultimoReporte = millis();

    Serial.print("Tiempo de ejecucion: ");
    Serial.print(millis() / 1000);
    Serial.print(" s | Total de interrupciones: ");
    Serial.println(contadorInterrupciones);
  }
}
```

La ISR alterna el estado del LED e incrementa el contador de interrupciones. El `loop()` aplica el estado al pin y muestra un reporte por serial cada tres segundos.

## Comparación de métodos de temporización

| Característica | `delay(ms)` | `millis()` | Timer de hardware |
|---|---|---|---|
| Bloqueo del programa | Sí, detiene el flujo principal. | No, permite continuar con otras tareas. | No, el conteo es independiente. |
| Precisión | Depende del flujo de ejecución. | Depende de la frecuencia con que se consulte. | Alta, según la configuración del temporizador. |
| Uso habitual | Pruebas sencillas. | Tareas periódicas controladas por software. | Muestreo, señales periódicas y control de motores. |

## Conclusiones

Los temporizadores permiten medir intervalos sin depender del programa principal. En el ESP32, el prescalador ajusta la frecuencia de conteo y la alarma define cuándo se produce una interrupción. El ejemplo muestra un evento periódico de un segundo mientras el `loop()` puede continuar con otras tareas.
