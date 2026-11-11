# Clase 3. Conjunto de instrucciones RISC y CISC y registros AVR

**Asignatura:** Sistemas Digitales (quinto ciclo, paralelo A)  
**Fecha:** Martes 13 de octubre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche  
**Estudiante:** Wilson Patricio Palma Samaniego  
**Objetivo:** Diferenciar RISC y CISC, reconocer los registros del ATmega328P y utilizar instrucciones básicas de ensamblador AVR.

## Conjunto de instrucciones: RISC y CISC

La ISA (*Instruction Set Architecture*) define las instrucciones que el software puede solicitar al procesador. Se revisaron dos enfoques: RISC y CISC.

| Característica | RISC | CISC |
|---|---|---|
| Diseño | Instrucciones sencillas y optimizadas. | Instrucciones que pueden realizar operaciones complejas. |
| Ejecución | Muchas instrucciones se completan en un ciclo. | Algunas instrucciones necesitan varios ciclos. |
| Memoria | En el modelo *Load-Store*, la ALU trabaja principalmente con registros. | Puede permitir operaciones que involucran directamente operandos en memoria. |
| Registros | Suele disponer de varios registros de propósito general. | La organización puede incluir registros más específicos. |
| Ejemplos | AVR, ARM y RISC-V. | x86 y x86-64. |

La sencillez de RISC facilita técnicas como el *pipelining*. En AVR se estudió una segmentación de dos etapas, búsqueda (*fetch*) y ejecución (*execute*), que permite solapar ambas actividades.

## Banco de registros del ATmega328P

El ATmega328P, de 8 bits, dispone de 32 registros generales (`R0` a `R31`) integrados en la CPU para guardar operandos y resultados temporales.

No todas las instrucciones pueden utilizar cualquier registro. Por ejemplo, `LDI` carga constantes únicamente en los registros altos (`R16` a `R31`). Además, los registros `R26` a `R31` se organizan en pares para formar tres punteros de 16 bits:

| Registros | Nombre | Uso |
|---|---|---|
| `R0` a `R15` | Registros bajos | Propósito general; no admiten `LDI` directamente. |
| `R16` a `R25` | Registros altos | Permiten cargar constantes con `LDI`. |
| `R26` y `R27` | X (`XL` y `XH`) | Puntero X de 16 bits. |
| `R28` y `R29` | Y (`YL` y `YH`) | Puntero Y de 16 bits. |
| `R30` y `R31` | Z (`ZL` y `ZH`) | Puntero Z de 16 bits. |

## Instrucciones básicas de ensamblador

**`LDI` (Load Immediate)** carga una constante de ocho bits en un registro entre `R16` y `R31`.

```assembly
LDI R16, 0xFF    ; Carga 255 en R16
LDI R17, 0x05    ; Carga 5 en R17
```

**`ADD` (Add)** suma dos registros y guarda el resultado en el primero. Por ejemplo:

```assembly
LDI R16, 10
LDI R17, 20
ADD R16, R17     ; R16 queda con el valor 30
```

**`SBI` y `CBI`** modifican un bit individual de un registro de entrada/salida. `SBI` lo pone en uno y `CBI` lo pone en cero. En el ejemplo de la sesión se utilizaron las siguientes instrucciones:

```assembly
SBI 0x04, 5      ; Pone en 1 el bit 5 de DDRB
SBI 0x05, 5      ; Pone en 1 el bit 5 de PORTB
```

La primera configura PB5 como salida y la segunda establece un nivel alto en ese pin.

## Inicialización y estado de los registros

No se debe asumir que todos los registros o variables contienen cero al iniciar. Usar un dato sin inicializar puede producir resultados inesperados.

```assembly
; El contenido de R17 no se ha definido en este ejemplo
LDI R16, 0x05
ADD R16, R17     ; El resultado depende del valor que tenga R17
```

Por ello, es necesario asignar valores conocidos a los registros y configurar los periféricos que se van a utilizar antes de ejecutar la lógica principal. Esta etapa de preparación evita depender de datos no establecidos por el programa.

## Ejecución continua en sistemas embebidos

En una computadora, un programa puede terminar y devolver el control al sistema operativo. En un microcontrolador, normalmente el programa permanece activo mientras el dispositivo esté encendido.

Después de un reinicio, el procesador comienza desde el vector de reset y ejecuta la inicialización. Luego entra en un bucle continuo, representado en C mediante `while(1)` o, en Arduino, mediante `loop()`. Dentro de ese ciclo se consultan entradas y se actualizan salidas.

```c
int main(void) {
    DDRB |= (1 << 5);   // Configura PB5 como salida

    while (1) {
        PORTB ^= (1 << 5);  // Invierte el estado del pin
        _delay_ms(500);
    }

    return 0; // No se alcanza mientras el bucle continúe
}
```

## Conclusión

La clase permitió relacionar la arquitectura del procesador con instrucciones concretas de ensamblador. Para programar el ATmega328P es necesario conocer la organización de sus registros, respetar las restricciones de cada instrucción e inicializar el hardware antes de utilizarlo. También se debe tener presente que un programa embebido normalmente mantiene un ciclo de ejecución continuo para responder a los cambios del entorno.
