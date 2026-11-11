# Clase 10: Interrupciones internas y externas, IRAM_ATTR y manejo de ISR

**Asignatura:** Sistemas Digitales  
**Fecha:** Viernes 30 de octubre y miércoles 4 de noviembre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche  
**Estudiante:** Wilson Patricio Palma Samaniego

## Objetivo de la clase

Diferenciar las interrupciones internas y externas del ESP32, comprender qué sucede cuando se ejecuta una rutina de servicio de interrupción (ISR) y reconocer la importancia de `IRAM_ATTR`, `volatile` y el patrón de banderas para comunicar una interrupción con el programa principal.

## ¿Qué es una interrupción?

Una interrupción permite atender un evento sin esperar a que `loop()` lo consulte. La CPU pausa temporalmente el programa, guarda el contexto necesario en la pila (*stack*), ejecuta la ISR (*Interrupt Service Routine*) y después continúa desde donde se detuvo.

Según su origen, pueden ser **externas**, cuando se producen por cambios eléctricos en los GPIO (botones, sensores o encoders), o **internas**, cuando las genera un periférico (alarma de timer, recepción UART, conversión ADC o Watchdog).

## Configuración de interrupciones externas en el ESP32

En el ESP32, un GPIO de entrada puede configurarse como fuente de interrupción mediante `attachInterrupt()`:

```cpp
attachInterrupt(digitalPinToInterrupt(pin), ISR_funcion, modo);
```

El parámetro `modo` indica qué cambio debe detectar el microcontrolador:

| Modo | Condición que genera la interrupción |
|---|---|
| `RISING` | Cambio de LOW a HIGH. |
| `FALLING` | Cambio de HIGH a LOW. |
| `CHANGE` | Cualquier cambio entre LOW y HIGH. |
| `LOW` | El pin permanece en nivel bajo. |
| `HIGH` | El pin permanece en nivel alto. |

Por ejemplo, con un botón configurado mediante `INPUT_PULLUP`, el estado normal es HIGH y al presionarlo cambia a LOW. En ese caso, se puede seleccionar `FALLING` para detectar la pulsación.

## Recomendaciones al programar una ISR

La ISR debe limitarse a registrar el evento. Si contiene demasiadas operaciones, puede retrasar otras tareas o provocar bloqueos.

Dentro de una ISR se deben evitar:

- Funciones de espera o bloqueo, como `delay()`, `delayMicroseconds()` y `yield()`.
- Operaciones lentas de entrada/salida, por ejemplo, `Serial.print()`, escritura en una tarjeta SD o actualización de una pantalla.
- Asignaciones de memoria y operaciones complejas con objetos o cadenas `String`.

Lo recomendable es mantener la rutina breve y utilizar el **patrón de bandera (*Flag Pattern*)**. La ISR activa una variable para indicar que ocurrió un evento, mientras que el programa principal se encarga de realizar las operaciones más pesadas.

## IRAM_ATTR y volatile

`IRAM_ATTR` indica que la función debe ubicarse en la memoria interna de instrucciones (IRAM), en lugar de depender de la Flash SPI. Esto es importante cuando la Flash está ocupada.

```cpp
void IRAM_ATTR miFuncionISR() {
    // Código breve de atención a la interrupción
}
```

`volatile` se aplica a variables que pueden cambiar desde una ISR. Así se evita que el compilador las trate como valores constantes y se asegura que sus lecturas consideren las modificaciones realizadas por la interrupción.

```cpp
volatile bool pulsoDetectado = false;
volatile uint32_t contadorEventos = 0;
```

## Ejemplo: conteo de pulsaciones con antirrebote

El siguiente código utiliza una interrupción externa en GPIO 4. La rutina registra los pulsos y activa una bandera; el `loop()` cambia el LED y muestra el conteo por el puerto serial.

```cpp
#include <Arduino.h>

const uint8_t PIN_PULSADOR = 4;
const uint8_t PIN_LED = 2;

volatile uint32_t contadorPulsos = 0;
volatile bool eventoPendiente = false;
volatile uint32_t ultimoTiempoInterrupcion = 0;

const uint32_t TIEMPO_DEBOUNCE_MS = 150;

void IRAM_ATTR ISR_ManejoPulsador() {
  uint32_t tiempoActual = millis();

  if (tiempoActual - ultimoTiempoInterrupcion > TIEMPO_DEBOUNCE_MS) {
    contadorPulsos++;
    eventoPendiente = true;
    ultimoTiempoInterrupcion = tiempoActual;
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(PIN_LED, OUTPUT);
  pinMode(PIN_PULSADOR, INPUT_PULLUP);

  attachInterrupt(
    digitalPinToInterrupt(PIN_PULSADOR),
    ISR_ManejoPulsador,
    FALLING
  );

  Serial.println("Sistema de interrupciones externas iniciado.");
}

void loop() {
  if (eventoPendiente) {
    eventoPendiente = false;

    digitalWrite(PIN_LED, !digitalRead(PIN_LED));

    Serial.print("Interrupcion detectada. Total de pulsos: ");
    Serial.println(contadorPulsos);
  }
}
```

La ISR solo registra el evento y activa `eventoPendiente`; la impresión por serial queda a cargo de `loop()`, fuera de la rutina.

## Comparación entre polling e interrupciones

| Criterio | Polling | Interrupción externa |
|---|---|---|
| Uso del procesador | Consulta repetidamente el pin. | Atiende el evento cuando se produce. |
| Tiempo de respuesta | Depende de la frecuencia de consulta del `loop()`. | Responde al evento configurado. |
| Pulsos rápidos | Pueden perderse si el programa está ocupado o bloqueado. | El hardware puede detectar cambios sin esperar a la consulta del programa. |
| Complejidad | Menor. | Requiere controlar variables compartidas y mantener la ISR breve. |

## Conclusiones

Las interrupciones permiten atender eventos internos y externos sin consultar constantemente los pines. Para utilizarlas correctamente, la ISR debe ser breve, `IRAM_ATTR` ubica la rutina en IRAM y `volatile` ayuda a manejar variables compartidas. El patrón de banderas separa la captura del evento de su procesamiento.
