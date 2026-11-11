# Cuaderno de Sistemas Digitales

Este repositorio reúne los apuntes de las 11 clases de **Sistemas Digitales**, correspondientes a la Unidad 1 de la carrera de Computación. La idea es tener en un solo lugar los temas revisados durante las sesiones, junto con explicaciones, tablas, ejemplos de código y preguntas que ayudan a repasar lo aprendido.

Los apuntes están organizados por fecha y se pueden consultar de manera individual desde el índice de esta página. Se trabajaron principalmente conceptos de arquitectura de microcontroladores, manipulación de registros, configuración de pines, temporización e interrupciones, utilizando como referencia el ATmega328P y el ESP32.

## Información de la asignatura

| Dato | Información |
|---|---|
| Asignatura | Sistemas Digitales |
| Carrera | Computación |
| Nivel | Quinto ciclo |
| Paralelo | A |
| Docente | Ing. Luis Sinche Cueva, Mg. |
| Estudiante | Wilson Patricio Palma Samaniego |
| Unidad | Microcontroladores para sistemas embebidos |
| Período de clases recopiladas | 30 de septiembre al 4 de noviembre de 2026 |

## Contenido del cuaderno

| Clase | Fecha | Tema | Apunte |
|---:|---|---|---|
| 01 | 30 de septiembre de 2026 | Del computador al microcontrolador | [Ver clase 1](docs/clase_01_30-sep_introduccion.md) |
| 02 | 7 de octubre de 2026 | Bits, operaciones bitwise y E/S mapeada en memoria | [Ver clase 2](docs/clase_02_07-oct_memorias_y_buses.md) |
| 03 | 13 de octubre de 2026 | Conjunto de instrucciones RISC y CISC; registros AVR | [Ver clase 3](docs/clase_03_13-oct_set_instrucciones.md) |
| 04 | 14 de octubre de 2026 | Operaciones bitwise y registros de entrada/salida | [Ver clase 4](docs/clase_04_14-oct_operaciones_bitwise.md) |
| 05 | 16 de octubre de 2026 | Estados de un pin digital y resistencia pull-up | [Ver clase 5](docs/clase_05_16-oct_estados_pin_pullup.md) |
| 06 | 20 de octubre de 2026 | GPIO Matrix, IO_MUX y pines de arranque del ESP32 | [Ver clase 6](docs/clase_06_20-oct_gpio_esp32_timers.md) |
| 07 | 21 de octubre de 2026 | Registros atómicos W1TS/W1TC e interrupciones | [Ver clase 7](docs/clase_07_21-oct_w1ts_w1tc_interrupciones.md) |
| 08 | 27 de octubre de 2026 | Tipos de datos, `millis()` y antirrebote | [Ver clase 8](docs/clase_08_27-oct_pines_y_timers_hw.md) |
| 09 | 28 de octubre de 2026 | Temporizadores de hardware en el ESP32 | [Ver clase 9](docs/clase_09_28-oct_temporizadores_hw.md) |
| 10 | 30 de octubre y 4 de noviembre de 2026 | Interrupciones internas y externas, `IRAM_ATTR` y manejo de ISR | [Ver clase 10](docs/clase_10_30-oct_interrupciones_in_ex.md) |
| 11 | 4 de noviembre de 2026 | Procesamiento de ISR y patrón de banderas | [Ver clase 11](docs/clase_11_04-nov_procesamiento_isr.md) |

## Temas que se trabajan

A lo largo de las sesiones se construye una visión general del funcionamiento de los microcontroladores y de la forma en que el software interactúa con el hardware. Los contenidos principales son:

- **Arquitectura y funcionamiento:** CPU, ALU, registros, ciclo de instrucción, buses y arquitecturas Harvard y Von Neumann.
- **Memoria y representación de datos:** Flash, SRAM, EEPROM, sistemas numéricos, bits y bytes.
- **Programación a bajo nivel:** conjunto de instrucciones AVR, operaciones bitwise y máscaras para modificar bits.
- **Entradas y salidas digitales:** registros `DDRx`, `PORTx` y `PINx`, estados de los pines y resistencias pull-up.
- **ESP32:** asignación de funciones mediante `GPIO Matrix` e `IO_MUX`, restricciones de GPIO y pines de arranque.
- **Temporización:** retardos, `millis()`, prescaladores, alarmas y temporizadores de hardware.
- **Atención de eventos:** polling, interrupciones, rutinas ISR, `IRAM_ATTR`, `volatile` y patrón de banderas.

## Organización de los archivos

Los apuntes están separados para que sea más sencillo consultar una sesión sin tener que recorrer un único documento extenso. La estructura del repositorio es la siguiente:

```text
cuaderno-sistemas-digitales/
├── README.md
└── docs/
    ├── clase_01_30-sep_introduccion.md
    ├── clase_02_07-oct_memorias_y_buses.md
    ├── clase_03_13-oct_set_instrucciones.md
    ├── clase_04_14-oct_operaciones_bitwise.md
    ├── clase_05_16-oct_estados_pin_pullup.md
    ├── clase_06_20-oct_gpio_esp32_timers.md
    ├── clase_07_21-oct_w1ts_w1tc_interrupciones.md
    ├── clase_08_27-oct_pines_y_timers_hw.md
    ├── clase_09_28-oct_temporizadores_hw.md
    ├── clase_10_30-oct_interrupciones_in_ex.md
    └── clase_11_04-nov_procesamiento_isr.md
```

## Cómo utilizar el cuaderno

Se recomienda revisar las clases en el orden presentado, ya que varios temas se apoyan en conocimientos de sesiones anteriores. Por ejemplo, las operaciones bitwise sirven para comprender la configuración de los registros de E/S; después, esos conocimientos se aplican al ESP32 y se complementan con temporizadores e interrupciones.

Cada archivo incluye el desarrollo del tema y ejemplos que permiten relacionar los conceptos con el código. Las tablas y preguntas de control también pueden utilizarse como material de repaso antes de una evaluación o práctica.

---

**Autor:** Wilson Patricio Palma Samaniego  
**Asignatura:** Sistemas Digitales
