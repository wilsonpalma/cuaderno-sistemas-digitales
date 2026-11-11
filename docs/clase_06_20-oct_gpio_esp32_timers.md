# Clase 6: Del AVR al ESP32: GPIO Matrix, IO_MUX y pines de arranque

**Asignatura:** Sistemas Digitales  
**Fecha:** Martes 20 de octubre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche

## Objetivo de la clase

Distinguir la asignación de pines del ATmega328P y del ESP32, comprender la función de `IO_MUX` y `GPIO Matrix`, reconocer las restricciones de algunos GPIO e introducir el uso de temporizadores por hardware.

## Diferencias en la asignación de pines

En el ATmega328P, las funciones de los pines están asociadas de forma más fija al hardware. Por ejemplo, PB5 corresponde al pin 13 de Arduino Uno y se relaciona con el LED integrado y la señal SCK del bus SPI.

El ESP32 ofrece mayor flexibilidad porque permite conectar sus periféricos internos a distintos pines físicos. Sin embargo, no todos los GPIO están disponibles para cualquier uso.

| Característica | ATmega328P | ESP32 |
|---|---|---|
| Asignación de funciones | Más fija por hardware. | Configurable mediante bloques internos. |
| Conexión de periféricos | Depende de pines asociados a funciones concretas. | Permite redirigir señales a GPIO disponibles. |

## Función de IO_MUX y GPIO Matrix

El enrutamiento del ESP32 se apoya en dos bloques:

- **`IO_MUX`:** selecciona la función del pin.
- **`GPIO Matrix`:** conecta señales de periféricos como UART, SPI, I2C o PWM con distintos GPIO.

El recorrido simplificado de una señal es:

```text
Periférico interno
       |
       v
  GPIO Matrix
       |
       v
    IO_MUX
       |
       v
   Pin físico GPIO
```

Así, un periférico puede utilizar diferentes conexiones físicas, siempre que se configure el GPIO y la función correspondiente.

## GPIO que requieren precaución

Algunos pines del ESP32 tienen restricciones. Los GPIO 34, 35, 36 (VP) y 39 (VN) funcionan solo como entradas; no pueden ser salidas ni disponen de pull-up o pull-down internas configurables.

Los GPIO 6 al 11 están conectados a la memoria Flash SPI que almacena el programa. Utilizarlos externamente puede interferir con su funcionamiento y provocar fallos o reinicios, por lo que deben evitarse.

## Voltaje de operación

El ESP32 trabaja con lógica de 3,3 V y sus GPIO no toleran señales de 5 V. Para conectarlo con dispositivos de ese voltaje se debe adaptar la señal mediante un divisor de tensión o un conversor de nivel lógico. En clase se tomó como referencia una corriente recomendada cercana a 12 mA por pin, por lo que no conviene alimentar directamente cargas elevadas desde un GPIO.

## Strapping pins y arranque

Los *strapping pins* son pines cuyo estado se revisa cuando el ESP32 se enciende o reinicia. Su nivel puede influir en el modo de arranque o en determinadas configuraciones del sistema.

| GPIO | Función mencionada en clase | Precaución |
|---|---|---|
| GPIO 0 | Selecciona entre descarga por UART y arranque normal. | Evitar un nivel incorrecto durante el encendido. |
| GPIO 2 | Participa en las condiciones de arranque y se relaciona con el LED de la placa. | Revisar su estado al programar o encender. |
| GPIO 12 (MTDI) | Interviene en la selección del voltaje de la memoria Flash. | Un nivel inadecuado puede impedir el arranque. |
| GPIO 15 (MTDO) | Se relaciona con los mensajes de depuración por UART0. | Su estado puede afectar la salida de esos mensajes. |

Por ello, antes de conectar sensores o pulsadores a estos pines, se debe revisar su función durante el arranque.

## Introducción a los temporizadores por hardware

También se comparó `delay()` con los temporizadores por hardware. `delay()` detiene el flujo principal durante el intervalo indicado, por lo que el programa no avanza con otras tareas.

Un temporizador es un periférico que cuenta pulsos del reloj independientemente de las instrucciones de la CPU. Puede utilizar un prescalador y generar un evento al alcanzar un valor determinado; este evento puede activar una interrupción. Mientras tanto, el procesador continúa atendiendo otras operaciones, como leer sensores o responder a botones.

## Conclusiones

El ESP32 permite distribuir las funciones de sus periféricos con mayor flexibilidad gracias a la GPIO Matrix y al IO_MUX. Sin embargo, es necesario respetar las restricciones de los pines de entrada, los utilizados por la memoria Flash y los que intervienen en el arranque. También se debe recordar que trabaja con señales de 3,3 V. Por último, los temporizadores por hardware permiten medir intervalos sin mantener bloqueada a la CPU, lo cual facilita la ejecución de varias tareas.
