# Clase 2. Bits, operaciones bitwise y entrada/salida mapeada en memoria

**Asignatura:** Sistemas Digitales (quinto ciclo, paralelo A)  
**Fecha:** Miércoles 7 de octubre de 2026  
**Docente:** Ing. Luis Sinche Cueva, Mg.  
**Estudiante:** Wilson Patricio Palma Samaniego  
**Objetivo:** Aplicar operaciones bit a bit para configurar y consultar los registros `DDR`, `PORT` y `PIN`.

## Representación de datos y niveles lógicos

En un circuito digital, `0` y `1` representan niveles de voltaje. El cero lógico es un nivel bajo, cercano a GND; el uno lógico es un nivel alto. El ATmega328P utiliza lógica de 5 V y el ESP32 de 3.3 V.

Un byte contiene ocho bits, con pesos que van desde 1 (bit 0) hasta 128 (bit 7).

| Posición | Bit 7 | Bit 6 | Bit 5 | Bit 4 | Bit 3 | Bit 2 | Bit 1 | Bit 0 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Peso | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

Por ejemplo, `0x81` equivale a `1000 0001` en binario y a 129 en decimal. En cambio, `0xFF` representa `1111 1111`, que corresponde a 255.

## Arquitecturas RISC y CISC

La ISA (*Instruction Set Architecture*) define las instrucciones de la CPU. En **RISC** suelen ser sencillas y requerir pocos ciclos; en **CISC** pueden ser más complejas y tardar varios ciclos.

Se mencionaron AVR, ARM, Xtensa y RISC-V como ejemplos RISC, y x86 como CISC. El ATmega328P es de 8 bits y el ESP32 de 32 bits; por ello, el primero necesita varias operaciones para procesar un dato de 32 bits.

## Flip-flops y registros de desplazamiento

Los registros digitales conservan bits mediante circuitos secuenciales llamados *flip-flops*. Se revisaron cuatro formas de entrada y salida en registros de desplazamiento:

- **SISO:** entrada serie y salida serie.
- **SIPO:** entrada serie y salida en paralelo.
- **PISO:** entrada en paralelo y salida serie.
- **PIPO:** entrada y salida en paralelo.

La transferencia en paralelo mueve varios bits al mismo tiempo; un bus de 8 bits, por ejemplo, transporta un byte completo.

## Entrada/salida mapeada en memoria

Los periféricos se controlan mediante registros ubicados en direcciones específicas de memoria o E/S. Al modificar uno de ellos puede cambiar el estado físico de un pin. Por ejemplo, `PORTB |= (1 << 5);` pone en uno el bit 5 de `PORTB`. El compilador traduce esta instrucción C a operaciones que el procesador puede ejecutar.

## Registros `DDR`, `PORT` y `PIN`

Cada puerto digital dispone de tres registros principales:

| Registro | Función | Bit en 0 | Bit en 1 |
|---|---|---|---|
| `DDRx` | Define la dirección del pin. | Entrada. | Salida. |
| `PORTx` | Escribe el nivel de salida o configura el pull-up en entrada. | Salida LOW / pull-up desactivado. | Salida HIGH / pull-up activado. |
| `PINx` | Lee el nivel eléctrico presente en el pin. | LOW. | HIGH. |

`PORTx` configura la salida o el pull-up; `PINx` permite leer el estado que llega físicamente al pin.

## Cuatro estados posibles de un pin

La combinación de `DDRx` y `PORTx` determina el comportamiento del pin:

| `DDRxn` | `PORTxn` | Estado | Descripción |
|---:|---:|---|---|
| 0 | 0 | Entrada flotante | Alta impedancia; puede captar ruido si no tiene referencia externa. |
| 0 | 1 | Entrada con pull-up | Mantiene el nivel alto mientras no se conecte a GND. |
| 1 | 0 | Salida LOW | El pin entrega un nivel bajo. |
| 1 | 1 | Salida HIGH | El pin entrega un nivel alto, según la alimentación. |

## Operaciones bitwise en C

Las máscaras permiten modificar un bit sin reemplazar el contenido completo del registro. La expresión `(1 << n)` desplaza el uno hasta la posición que se desea seleccionar.

```c
PORTB |= (1 << 5);    // SET: pone el bit 5 en uno
PORTB &= ~(1 << 4);   // CLEAR: pone el bit 4 en cero
PORTB ^= (1 << 5);    // TOGGLE: invierte el bit 5

if (PIND & (1 << 2)) { // CHECK: comprueba si el bit 2 vale uno
    // El bit se encuentra en HIGH
}
```

En síntesis, `|=` activa, `&= ~` limpia, `^=` invierte y `&` comprueba un bit. Así se evita modificar los demás bits del puerto.

## Preguntas de control y conclusión

**¿Cuál es el equivalente binario de `0x81`?** `1000 0001`.

**¿Cómo se apaga el bit 4 de `PORTB`?** Con `PORTB &= ~(1 << 4);`.

**¿Qué se lee al presionar un botón conectado a GND si está activa la resistencia pull-up?** Se lee `0` lógico, porque el botón conecta el pin con tierra.

Los registros permiten controlar directamente el hardware. Para utilizarlos correctamente hay que distinguir sus funciones y aplicar máscaras al modificar bits individuales.
