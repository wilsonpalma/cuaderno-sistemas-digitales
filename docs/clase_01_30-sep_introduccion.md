# Clase 1. Del computador al microcontrolador

**Asignatura:** Sistemas Digitales (quinto ciclo, paralelo A)  
**Fecha:** Miércoles 30 de septiembre de 2026  
**Docente:** Ing. Luis Sinche Cueva, Mg.  
**Estudiante:** Wilson Patricio Palma Samaniego  
**Objetivo:** Identificar los bloques principales de un microcontrolador y distinguirlo de un microprocesador.

## Sistemas embebidos

Un sistema embebido es un dispositivo diseñado para realizar una tarea específica dentro de un equipo. No busca ejecutar todo tipo de programas, como una computadora personal, sino cumplir una función concreta de forma continua. Por eso suele trabajar con recursos limitados de memoria y procesamiento, y debe responder a las señales que recibe.

Se encuentran en lavadoras, routers, termostatos y automóviles, donde reciben información y controlan componentes.

## Microprocesador y microcontrolador

La diferencia está en los componentes integrados: el microprocesador contiene principalmente la CPU y requiere memorias y periféricos externos; el microcontrolador reúne estos elementos en un solo chip.

| Criterio | Microprocesador (µP) | Microcontrolador (MCU) |
|---|---|---|
| Componentes | CPU; requiere memoria y E/S externas. | CPU, memorias y periféricos integrados. |
| Uso habitual | Computadoras y servidores. | Control de tareas específicas. |
| Consumo y costo | Generalmente mayores. | Normalmente menores. |
| Frecuencia | Puede trabajar en GHz. | Suele utilizar frecuencias más moderadas. |

## CPU y ciclo de instrucción

La CPU ejecuta las instrucciones del programa. La **ALU** realiza operaciones matemáticas y lógicas; los **registros** guardan datos temporales; la **Unidad de Control** interpreta y coordina las instrucciones; y el **Contador de Programa (PC)** indica cuál se ejecutará después. El AVR del ATmega328P dispone de 32 registros generales (`r0` a `r31`).

El procesamiento sigue un ciclo que se repite: **buscar** la instrucción en la memoria de programa (*fetch*), **decodificarla** (*decode*) y **ejecutarla** (*execute*). El reloj del sistema marca el ritmo. Para un ATmega328P a 16 MHz, el periodo de cada pulso es:

\[
T = \frac{1}{16\,000\,000} = 62.5\text{ ns}
\]

## Buses de comunicación

Los componentes internos se comunican mediante tres buses: direcciones indica dónde se trabajará, datos transporta la información y control señala si se realizará una lectura o escritura.

| Bus | Función | Flujo habitual |
|---|---|---|
| Direcciones | Identifica una dirección de memoria o periférico. | CPU hacia memoria o periférico. |
| Datos | Transporta la información. | Bidireccional. |
| Control | Indica la operación que se debe realizar. | Según la señal de control. |

## Arquitecturas Von Neumann y Harvard

En **Von Neumann**, las instrucciones y los datos comparten una misma memoria y un mismo bus. Esto puede generar un cuello de botella porque ambos tipos de acceso utilizan el mismo camino.

En **Harvard**, la memoria y los buses del programa están separados de los destinados a los datos. Así, la CPU puede buscar una instrucción mientras trabaja con información. En la sesión se mencionó a los microcontroladores ATmega328P y ESP32 como ejemplos de esta arquitectura.

## Memorias principales

La **Flash** conserva el programa; la **SRAM** guarda variables temporales y pierde su contenido al apagar el equipo; la **EEPROM** conserva parámetros después de un reinicio.

| Memoria | Volátil | Función | ATmega328P | ESP32 |
|---|---|---|---:|---:|
| Flash | No | Guarda el programa. | 32 KB | 4 MB (SPI externa, según la placa) |
| SRAM | Sí | Almacena datos de trabajo. | 2 KB | Aproximadamente 520 KB |
| EEPROM | No | Conserva configuraciones. | 1 KB | Se emula en Flash; no tiene EEPROM física. |

La EEPROM admite una cantidad limitada de escrituras. Por eso no conviene actualizarla constantemente desde el bucle principal, ya que se reduce su vida útil.

## ATmega328P frente a ESP32

| Parámetro | ATmega328P (Arduino Uno) | ESP32 DevKit V1 |
|---|---|---|
| CPU | AVR de 8 bits, un núcleo | Xtensa LX6 de 32 bits, dos núcleos |
| Frecuencia | 16 MHz | Hasta 240 MHz |
| Nivel lógico | 5 V | 3.3 V; no tolera directamente 5 V |
| Conectividad | Sin Wi-Fi ni Bluetooth integrados | Wi-Fi y Bluetooth |
| Pines | Asignación ligada al hardware | GPIO Matrix e IO_MUX permiten mayor flexibilidad |

## Preguntas de control y conclusión

**¿Qué transportan los buses?** Direcciones identifica la ubicación; datos lleva la información; y control indica la operación.

**¿Dónde se guardan las variables durante la ejecución?** En la SRAM, que almacena los datos temporales.

**¿Qué ventaja tiene Harvard?** Al separar los accesos al programa y a los datos, permite que la CPU busque instrucciones mientras trabaja con información.

En conclusión, un microcontrolador integra procesamiento, memoria y periféricos para controlar tareas concretas. Conocer la función de cada bloque permite comprender mejor cómo se ejecutan los programas y cómo se relaciona el código con el hardware.
