# Clase 5: Estados de un pin digital y resistencia pull-up

**Asignatura:** Sistemas Digitales  
**Fecha:** Viernes 16 de octubre de 2026  
**Unidad:** Microcontroladores para sistemas embebidos  
**Docente:** Ing. Luis Sinche

## Objetivo de la clase

Comprender cómo se combinan los registros `DDRx` y `PORTx` para configurar un pin digital, reconocer la utilidad de la resistencia pull-up interna y reforzar estos conceptos mediante preguntas de control.

## Combinaciones de DDR y PORT

En el ATmega328P, el modo de funcionamiento de un pin depende de dos bits: el correspondiente a `DDRx` y el de `PORTx`. El primero define si el pin es entrada o salida; el segundo establece el nivel de salida o permite activar la resistencia pull-up cuando se trabaja como entrada.

| `DDRxn` | `PORTxn` | Estado del pin | Comportamiento |
|:---:|:---:|---|---|
| `0` | `0` | Entrada flotante | Entrada en alta impedancia, sin pull-up interna. |
| `0` | `1` | Entrada con pull-up | La resistencia interna mantiene normalmente la lectura en `1`. |
| `1` | `0` | Salida LOW | El pin entrega un nivel bajo, cercano a GND. |
| `1` | `1` | Salida HIGH | El pin entrega un nivel alto, cercano a VCC. |

## El estado flotante y su solución

Cuando un pin se configura con `DDR = 0` y `PORT = 0`, queda como entrada en alta impedancia. Si no existe una conexión externa que determine su nivel, puede captar interferencias del entorno. Por ello, el valor leído podría cambiar entre cero y uno aunque no se esté accionando ningún componente. A este comportamiento se lo conoce como estado flotante.

Para evitarlo se puede activar la resistencia pull-up interna. En el ATmega328P se configura el pin como entrada y se coloca en uno el bit correspondiente de `PORTx`:

```c
DDRB &= ~(1 << PB2);  // PB2 como entrada
PORTB |= (1 << PB2);  // Activa la resistencia pull-up
```

La resistencia conecta débilmente la entrada con VCC. Su valor es aproximadamente de 20 a 50 kΩ. Si el pulsador permanece abierto, el pin se mantiene en nivel alto y se lee un `1`. Cuando se presiona un botón conectado a GND, la entrada queda conectada a tierra y se lee un `0`.

Esta configuración evita tener que colocar una resistencia pull-up externa en cada entrada, siempre que las características del circuito permitan utilizar la interna. También implica una lógica que puede parecer invertida al principio: botón suelto equivale a `1` y botón presionado equivale a `0`.

## Lectura del estado mediante PINx

Los registros `PORTx` y `PINx` no deben confundirse. El primero se utiliza para establecer el nivel de salida o activar la pull-up; el segundo permite consultar el nivel que está presente en el pin físico.

El siguiente ejemplo detecta la pulsación de un botón conectado a PB2:

```c
DDRB &= ~(1 << PB2);   // Configura PB2 como entrada
PORTB |= (1 << PB2);   // Activa la pull-up

if (!(PINB & (1 << PB2))) {
    // El pulsador está presionado
}
```

La negación `!` permite comprobar el nivel bajo. Esto se debe a que, con la conexión pull-up a GND, el estado activo del botón corresponde a cero lógico.

## Preguntas de control

**¿Cuánto vale `0x81` en binario?**

El dígito `8` equivale a `1000` y el dígito `1` equivale a `0001`. Por tanto, `0x81 = 1000 0001`; los bits de las posiciones 7 y 0 están encendidos.

**¿Cómo se apaga únicamente el bit 4 de `PORTB`?**

Se utiliza la operación CLEAR:

```c
PORTB &= ~(1 << 4);
```

La expresión `1 << 4` produce `00010000`. Después, `~` invierte la máscara y AND fuerza a cero el bit 4, mientras conserva los demás.

**Con `DDRD2 = 0` y `PORTD2 = 1`, ¿qué lee `PIND2` al presionar un pulsador conectado a GND?**

Lee `0`. El pin está configurado como entrada y tiene activada la pull-up. Al presionar el botón, queda conectado a tierra y su nivel pasa a LOW.

## Conclusiones

La combinación de `DDR` y `PORT` permite establecer cuatro estados para un pin digital. La resistencia pull-up ayuda a evitar lecturas inestables en entradas que utilizan pulsadores. Para trabajar correctamente con estas conexiones, hay que recordar que `PORTx` configura la salida o la resistencia, mientras que `PINx` se consulta para conocer el estado real de la entrada.
