# Clase 8: Tipos de datos, temporización con millis() y lectura de pulsadores

**Asignatura:** Sistemas Digitales  
**Fecha:** Martes 27 de octubre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche

## Objetivo de la clase

Utilizar tipos de datos de tamaño definido para trabajar con tiempos y contadores, sustituir retardos bloqueantes por una temporización basada en `millis()` y reconocer los cambios de estado de un pulsador mediante detección de flancos y antirrebote.

## Tipos de datos con tamaño definido

El tamaño de `int` cambia según la arquitectura: en el ATmega328P tiene 16 bits y en el ESP32, 32 bits. Para evitar diferencias se puede utilizar `<stdint.h>`, que ofrece tipos de tamaño definido:

| Tipo | Rango sin signo | Rango con signo | Memoria |
|---|---:|---:|---:|
| `uint8_t` / `int8_t` | 0 a 255 | -128 a 127 | 1 byte |
| `uint16_t` / `int16_t` | 0 a 65 535 | -32 768 a 32 767 | 2 bytes |
| `uint32_t` / `int32_t` | 0 a 4 294 967 295 | -2 147 483 648 a 2 147 483 647 | 4 bytes |

Un problema que puede presentarse es el desbordamiento (*overflow*). Por ejemplo, si `uint8_t contador = 255` y se incrementa una unidad, el valor vuelve a cero. Por eso, una variable de 8 bits no es adecuada para guardar tiempos que pueden crecer durante mucho tiempo.

## Temporización no bloqueante con millis()

`millis()` devuelve los milisegundos transcurridos desde que se encendió el microcontrolador. Se almacena normalmente en 32 bits y se desborda aproximadamente cada 49,7 días.

En cambio, `delay()` detiene el flujo principal durante el intervalo indicado:

```c
void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(1000);
  digitalWrite(LED_PIN, LOW);
  delay(1000);
}
```

Durante esos retardos, el flujo principal no puede continuar con las demás instrucciones que estén después de la llamada. Para evitarlo se puede comparar el tiempo actual con una marca de tiempo anterior:

```c
#include <stdint.h>

const uint16_t INTERVALO_LED = 1000;
uint32_t tiempoAnterior = 0;
uint8_t estadoLED = 0;

void loop() {
  uint32_t tiempoActual = millis();

  if (tiempoActual - tiempoAnterior >= INTERVALO_LED) {
    tiempoAnterior = tiempoActual;
    estadoLED = !estadoLED;
    digitalWrite(LED_PIN, estadoLED);
  }

  // Aquí pueden continuar otras tareas.
}
```

La resta `tiempoActual - tiempoAnterior` permite comprobar cuánto tiempo ha pasado sin depender de que el contador de `millis()` mantenga siempre un valor creciente. Por eso, este patrón es preferible a comparar directamente con una suma de tiempos.

## Lectura por nivel y detección de flancos

La lectura por nivel consulta continuamente si el botón está en `LOW` o `HIGH`; si permanece presionado, una acción podría repetirse. La detección de flancos reconoce el instante en que cambia la señal:

- **Falling Edge:** transición de HIGH a LOW. Con una pull-up, suele corresponder al momento de presionar el botón.
- **Rising Edge:** transición de LOW a HIGH. En la misma conexión, corresponde normalmente al momento de soltarlo.

Un ejemplo de detección de flanco de bajada es:

```c
uint8_t estadoAnteriorBoton = HIGH;

void evaluarBoton() {
  uint8_t estadoActualBoton = digitalRead(BOTON_PIN);

  if (estadoAnteriorBoton == HIGH && estadoActualBoton == LOW) {
    alternarModo(); // Se detectó una nueva pulsación
  }

  estadoAnteriorBoton = estadoActualBoton;
}
```

## Antirrebote por software

Los contactos de un pulsador pueden producir oscilaciones eléctricas durante los primeros milisegundos. El microcontrolador podría interpretarlas como varias pulsaciones; este fenómeno se conoce como rebote mecánico. Para reducirlo se registra el cambio con `millis()` y se espera a que transcurra un intervalo antes de aceptar el nuevo estado.

```c
uint32_t ultimoTiempoRebote = 0;
const uint8_t TIEMPO_DEBOUNCE = 50;
uint8_t ultimoEstadoEstable = HIGH;

void lecturaBotonConDebounce() {
  uint8_t lecturaActual = digitalRead(BOTON_PIN);

  if (lecturaActual != estadoAnteriorBoton) {
    ultimoTiempoRebote = millis();
  }

  if ((millis() - ultimoTiempoRebote) > TIEMPO_DEBOUNCE) {
    if (lecturaActual != ultimoEstadoEstable) {
      ultimoEstadoEstable = lecturaActual;

      if (ultimoEstadoEstable == LOW) {
        ejecutarAccion();
      }
    }
  }

  estadoAnteriorBoton = lecturaActual;
}
```

En el ejemplo se utiliza un intervalo de 50 ms para filtrar los cambios producidos por el contacto mecánico.

## Conclusiones

Los tipos de tamaño definido permiten controlar mejor los valores y la memoria utilizada. `millis()` ayuda a organizar tareas sin detener el programa, mientras que la detección de flancos y el antirrebote permiten interpretar las pulsaciones de forma más estable.
