# Clase 7: Registros atómicos W1TS y W1TC e introducción a las interrupciones

**Asignatura:** Sistemas Digitales  
**Fecha:** Miércoles 21 de octubre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche

## Objetivo de la clase

Entender por qué algunas operaciones sobre los registros pueden generar conflictos cuando existen interrupciones o más de un núcleo trabajando, conocer los registros atómicos `GPIO_OUT_W1TS` y `GPIO_OUT_W1TC` del ESP32 y distinguir entre la lectura por sondeo (*polling*) y las interrupciones.

## El problema de Read-Modify-Write

En el ATmega328P, una operación para cambiar un bit de un puerto normalmente implica tres pasos, conocidos como **Read-Modify-Write (R-M-W)**:

1. **Read:** la CPU lee el contenido completo del registro.
2. **Modify:** modifica el bit necesario mediante una máscara lógica, por ejemplo, con OR o AND.
3. **Write:** escribe nuevamente el valor actualizado en el registro.

El inconveniente aparece cuando otro proceso intenta modificar el mismo registro justo entre la lectura y la escritura. Puede ocurrir, por ejemplo, si se ejecuta una interrupción o si el segundo núcleo del ESP32 modifica otro pin del mismo puerto. En ese caso, la tarea principal puede terminar escribiendo un valor anterior y sobrescribir el cambio realizado por la otra ejecución. A este conflicto se lo conoce como **condición de carrera (*race condition*)**.

## Escrituras atómicas en el ESP32: W1TS y W1TC

El ESP32 dispone de registros específicos que permiten modificar las salidas sin tener que leer primero el contenido completo del puerto. Se los conoce como registros de escritura de un uno para establecer o limpiar bits:

| Registro | Nombre | Funcionamiento |
|---|---|---|
| `GPIO_OUT_W1TS` | *Write 1 to Set* | Al escribir `1` en una posición, establece esa salida en nivel alto. Los bits escritos como `0` no cambian. |
| `GPIO_OUT_W1TC` | *Write 1 to Clear* | Al escribir `1` en una posición, coloca esa salida en nivel bajo. Los bits escritos como `0` no cambian. |

Un ejemplo permite comparar ambas formas de trabajo:

```c
// Forma tradicional: lectura, modificación y escritura
GPIO.out |= (1 << 4);   // Establece GPIO 4 en alto
GPIO.out &= ~(1 << 4);  // Coloca GPIO 4 en bajo

// Escritura mediante registros atómicos
GPIO.out_w1ts = (1 << 4); // Establece GPIO 4 en alto
GPIO.out_w1tc = (1 << 4); // Coloca GPIO 4 en bajo
```

Hay que tener presente una diferencia importante: con `W1TS` y `W1TC` se utiliza asignación directa (`=`), no operadores como `|=` o `&=`. El propio hardware se encarga de modificar los bits indicados, mientras que las posiciones escritas con cero permanecen sin cambios.

## Polling frente a interrupciones

Para detectar situaciones como la pulsación de un botón existen dos enfoques. El primero es el *polling* o sondeo, que consiste en consultar repetidamente el estado del pin desde el programa principal. Es sencillo de implementar, pero consume tiempo de procesamiento y puede pasar por alto una señal breve, especialmente cuando existe un `delay()`.

El segundo enfoque utiliza interrupciones. En este caso, un evento detectado por el hardware hace que la CPU pause temporalmente el flujo principal y ejecute una rutina especial llamada ISR (*Interrupt Service Routine*). Después de atender el evento, el programa puede continuar con su ejecución normal.

| Criterio | Polling | Interrupciones |
|---|---|---|
| Uso de CPU | Se consulta el estado constantemente. | La CPU atiende el evento cuando ocurre. |
| Respuesta | Depende de cuándo se vuelva a consultar el pin. | Se activa por el evento configurado en el hardware. |
| Eventos rápidos | Pueden pasar inadvertidos durante un retardo. | El hardware puede detectar el cambio aunque el programa principal esté ocupado. |
| Programación | Es más sencilla. | Requiere cuidado con las variables compartidas y la concurrencia. |

## Conclusiones

Las operaciones R-M-W pueden provocar condiciones de carrera cuando una interrupción u otro núcleo modifica el mismo registro durante el proceso. Los registros `W1TS` y `W1TC` del ESP32 permiten establecer o limpiar salidas sin realizar esa lectura previa. Por otra parte, el *polling* y las interrupciones resuelven la detección de eventos de maneras distintas: el primero consulta continuamente, mientras que las segundas permiten reaccionar cuando el hardware detecta un evento.
