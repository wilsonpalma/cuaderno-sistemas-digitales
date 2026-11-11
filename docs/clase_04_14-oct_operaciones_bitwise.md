# Clase 4: Operaciones bitwise y registros de entrada/salida

**Asignatura:** Sistemas Digitales  
**Fecha:** Miércoles 14 de octubre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche

## Objetivo de la clase

Aplicar operaciones bit a bit para modificar registros, diferenciar las funciones de `DDRx`, `PORTx` y `PINx`, y relacionar el código C con las instrucciones que ejecuta el microcontrolador.

## Manipulación de bits mediante máscaras

Los registros de los puertos contienen varios bits y cada uno puede estar relacionado con un pin físico. Si se desea cambiar un solo pin, no conviene reemplazar todo el contenido del registro con una asignación directa, porque también podrían modificarse los demás pines. Para evitarlo se utilizan máscaras.

La expresión `(1 << n)` genera un valor en el que únicamente está encendido el bit de la posición `n`. Con esa máscara se realizan cuatro operaciones habituales:

| Operación | Operador | Función | Ejemplo |
|---|---|---|---|
| SET | `\|` | Coloca el bit indicado en `1`. | `REG \|= (1 << n);` |
| CLEAR | `&` y `~` | Coloca el bit indicado en `0`. | `REG &= ~(1 << n);` |
| TOGGLE | `^` | Invierte el estado del bit. | `REG ^= (1 << n);` |
| CHECK | `&` | Comprueba el estado del bit. | `REG & (1 << n)` |

Por ejemplo, para configurar PB5 como salida se escribe:

```c
DDRB |= (1 << 5);
```

La máscara corresponde a `00100000`. La operación OR coloca el bit 5 en uno y conserva los demás. Para apagar el bit 4 de `PORTB` se utiliza `PORTB &= ~(1 << 4);`. En este caso, NOT invierte la máscara y AND permite borrar únicamente la posición seleccionada.

Para alternar PB5 se puede usar `PORTB ^= (1 << 5);`: si estaba en cero pasa a uno y, si estaba en uno, pasa a cero. Por otro lado, una condición como `if (PIND & (1 << 2))` comprueba si el bit 2 está en nivel alto. Con un pulsador conectado a GND y una pull-up activa, se puede utilizar `if (!(PIND & (1 << 2)))` para detectar que fue presionado.

## Función de los registros DDR, PORT y PIN

Cada puerto digital del ATmega328P dispone de tres registros principales:

| Registro | Función |
|---|---|
| `DDRx` | Define la dirección del pin: `0` para entrada y `1` para salida. |
| `PORTx` | En salida establece LOW o HIGH. En entrada, un `1` activa la resistencia pull-up. |
| `PINx` | Permite consultar el nivel lógico que llega físicamente al pin. |

Por tanto, `DDR` indica cómo se utilizará el pin, `PORT` establece su salida o configura la pull-up y `PIN` permite leer el estado eléctrico de la entrada. Esta distinción es necesaria al trabajar con botones y sensores.

## Del código C al ensamblador

El código escrito en C pasa por un proceso de compilación. Por ejemplo, una instrucción como:

```c
PORTB |= (1 << 5);
```

puede traducirse a una instrucción AVR como:

```asm
sbi 0x05, 5
```

`SBI` (*Set Bit in I/O Register*) coloca en uno un bit de un registro de entrada/salida. En el ATmega328P, `PORTB` corresponde a la dirección de E/S `0x05` y el bit modificado es el 5. Para borrar un bit existe la instrucción `CBI`. Así se observa cómo una operación de C se convierte en instrucciones específicas del procesador.

## Ejercicios revisados

**Convertir `0x81` a binario.** El dígito hexadecimal `8` equivale a `1000` y el `1` equivale a `0001`. Por lo tanto, `0x81 = 1000 0001`.

**Apagar únicamente el bit 4 de `PORTB`.**

```c
PORTB &= ~(1 << 4);
```

**¿Qué lee `PIND2` al presionar un pulsador conectado a GND?** Si `DDRD2 = 0`, PD2 es una entrada; si `PORTD2 = 1`, se activa la pull-up. Al estar suelto el botón se lee normalmente `1`; al presionarlo, el pin se conecta a tierra y se lee `0`.

## Conclusiones

Las máscaras permiten modificar bits específicos sin alterar innecesariamente el resto del puerto. También es importante distinguir los registros: `DDR` configura la dirección, `PORT` controla la salida o la pull-up y `PIN` permite consultar la entrada. Comprender la traducción de C a ensamblador ayuda a relacionar el programa con el funcionamiento del microcontrolador.
