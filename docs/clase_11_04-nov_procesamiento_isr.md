# Clase 11: Procesamiento de ISR, patrón de banderas y repaso de la Unidad 1

**Asignatura:** Sistemas Digitales (5.º ciclo, paralelo A)  
**Fecha:** Miércoles 4 de noviembre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche

## Objetivo de la clase

Profundizar en el uso conjunto de interrupciones y temporizadores de hardware, aplicar el patrón de banderas para dejar el procesamiento más complejo fuera de una ISR y relacionar los temas estudiados durante la Unidad 1.

## Manejo de eventos: polling, ISR directa y patrón de banderas

En un sistema embebido se puede detectar un evento mediante *polling*, consultando continuamente el pin desde `loop()`, o mediante una ISR. Ejecutar muchas operaciones dentro de la ISR puede retrasar otras tareas. Por eso, en clase se destacó el **patrón de banderas**:

| Criterio | Polling | ISR con procesamiento extenso | ISR + patrón de banderas |
|---|---|---|---|
| Uso de CPU | Consulta continuamente el evento. | Atiende el evento, pero mantiene ocupada la CPU durante la rutina. | La ISR registra el evento y el programa principal realiza el resto. |
| Respuesta | Depende de la frecuencia de consulta. | Se ejecuta al ocurrir la interrupción. | La captura se realiza por hardware y el procesamiento continúa en `loop()`. |
| Organización | Código sencillo, pero con consultas constantes. | Puede acumular demasiadas operaciones dentro de la ISR. | Separa la captura del evento y su procesamiento. |

## Funcionamiento del patrón de banderas

El patrón tiene dos fases. En la **captura**, la ISR registra el evento, actualiza el contador y activa una variable booleana `volatile`. En el **procesamiento**, `loop()` detecta la bandera, la limpia y realiza las tareas más largas, como imprimir por serial o actualizar una pantalla. Así, la ISR se mantiene breve.

## Ejemplo de implementación

El siguiente programa utiliza GPIO 4 como entrada con resistencia pull-up y GPIO 2 como salida. Al detectar una pulsación, la ISR incrementa el contador y activa una bandera. Después, el programa principal lee los datos y realiza la acción correspondiente.

```cpp
#include <Arduino.h>

const uint8_t PIN_SENSOR = 4;
const uint8_t PIN_ACTUADOR = 2;

volatile bool flagEvento = false;
volatile uint32_t contadorPulsos = 0;
volatile uint32_t ultimoTiempoISR = 0;

const uint32_t TIEMPO_DEBOUNCE_MS = 200;

void IRAM_ATTR isrManejadorEvento() {
    uint32_t tiempoActual = millis();

    if (tiempoActual - ultimoTiempoISR > TIEMPO_DEBOUNCE_MS) {
        contadorPulsos++;
        flagEvento = true;
        ultimoTiempoISR = tiempoActual;
    }
}

void setup() {
    Serial.begin(115200);
    pinMode(PIN_ACTUADOR, OUTPUT);
    pinMode(PIN_SENSOR, INPUT_PULLUP);

    attachInterrupt(
        digitalPinToInterrupt(PIN_SENSOR),
        isrManejadorEvento,
        FALLING
    );

    Serial.println("Sistema inicializado: patron de banderas activo");
}

void loop() {
    if (flagEvento) {
        // Sección crítica: copiar los datos compartidos
        noInterrupts();
        flagEvento = false;
        uint32_t copiasPulsos = contadorPulsos;
        interrupts();

        // Procesamiento fuera de la ISR
        digitalWrite(PIN_ACTUADOR, HIGH);

        Serial.print("Evento detectado. Pulsos acumulados: ");
        Serial.println(copiasPulsos);

        delay(50); // Simula una tarea de mayor duración
        digitalWrite(PIN_ACTUADOR, LOW);
    }

    // Aquí pueden ejecutarse otras tareas del programa.
}
```

La ISR registra la pulsación y activa la bandera. En `loop()` se deshabilitan brevemente las interrupciones para limpiar esa bandera y copiar el contador; después se habilitan nuevamente y se realiza el procesamiento.

## Repaso general de la Unidad 1

Durante la unidad se estudiaron los componentes internos de un microcontrolador y la forma de interactuar con sus periféricos. Los temas se relacionan entre sí de la siguiente manera:

| Área | Contenidos principales |
|---|---|
| Arquitectura de CPU | ALU, registros de propósito general, contador de programa y unidad de control. |
| Memorias y buses | Flash, SRAM, EEPROM; buses de direcciones, datos y control; arquitectura Harvard. |
| Entrada y salida en AVR | Registros `DDRx`, `PORTx` y `PINx`, junto con operaciones bitwise. |
| Entrada y salida en ESP32 | `IO_MUX`, `GPIO Matrix` y registros atómicos `W1TS` y `W1TC`. |
| Temporización | `millis()`, prescaladores, temporizadores de hardware y alarmas. |
| Interrupciones | Eventos externos e internos, ISR, `IRAM_ATTR`, `volatile` y patrón de banderas. |

Estos contenidos muestran cómo se pasa de la arquitectura de la CPU al control de pines físicos y, finalmente, a la atención de eventos mediante temporizadores e interrupciones.

## Preguntas de autoevaluación

**¿Por qué una variable compartida entre una ISR y el `loop()` debe declararse con `volatile`?**

Porque la ISR puede modificarla en cualquier momento. `volatile` evita que el compilador asuma que su valor no cambia y permite que el programa principal lea las actualizaciones.

**¿Qué puede suceder al ejecutar `Serial.println()` dentro de una ISR en el ESP32?**

Puede provocar bloqueos, ya que la comunicación serial utiliza buffers y mecanismos internos que no conviene ejecutar desde una ISR. La impresión debe realizarse desde `loop()`.

## Conclusión de la Unidad 1

La unidad integra arquitectura, registros, temporización e interrupciones. El patrón de banderas resume su aplicación: la ISR registra el evento y `loop()` lo procesa.
