
# Low-level-data format

Substrate códifica los datos en el formato SCALE (Simple Concatenated Aggregate Little-Endian), tal como fue implementado por:
parity-scale-codec en Rust
@polkadot/types en Javascript
ChainSafe/gossamer en Ir
opennetsys/golkadot en Ir
soramitsu/kagome in C++
polkascan/py-scale-codec en Python
Es un formato de codificación de datos extremadamente ligero, libre de copias de codificación y permite la decodificación de datos en entornos con recursos limitados o restrictivos, como es el caso del "runtime" de Substrate. No describe una forma particular de codificar los datos, sino que asume que, donde se decodifican los datos se dispone del conocimiento acerca de los tipos que han sido codificados.
La representación codificada está definida de la siguiente manera:
Enteros de tamaño fijo
Los enteros básicos (Basic Integers) están codificados usando un tamaño fijo y el fomato de Little-Endian(LE). Por ejemplo.
Con signo 8-bit integer 69: 0x45
Sin signo 16-bit integer 42: 0x2a00
Sin signo 32-bit integer 16777215: 0xffffff00
Boolean
Los valores "booleanos" (Boolean) están codificados usando el bit menos significativo (LSB) de un solo byte:
boolean false: 0x00
boolean true: 0x01
Enteros compactos/generales
Un entero compacto o general (general/compact Integer) es suficiente para codificar enteros grandes(hasta 2**536 bits) y es más eficiente a la hora de codificar valores que la versión de tamaño fijo. (Aunque para valores de un solo Byte, el entero de tamaño fijo nunca es peor.)
Está codificado con los dos bits menos significativos, los cuáles indican el modo:
0b00: modo de un solo byte (single-byte mode); seis bits superiores son la codificación LE(Little Endian) del valor (solo válido para valores comprendidos entre 0-63).
0b01: modo de dos bytes (two-byte mode): los seis bits superiores y el siguiente byte son la codificación LE del valor (válido solo para valores comprendidos entre 64 - (2**14-1)).
0b10: modo de 4-bytes (four-byte mode): los 6 bits superiores y los siguientes 3 bytes son la codificación LE del valor (solo válido para valores entre (2**14-1) - (2**30-1)).
0b11: modo entero grande (Big-integer mode): Los seis bits superiores son el número de bytes siguiente, menos cuatro. El valor está contenido, codificado en LE, en los bytes siguientes. El byte final (El más significativo) debe ser distinto de cero (non-zero). Sólo válido para valores comprendidos entre (2**30-1) - (2**536-12).
Ejemplos:
sin signo integer 0: 0x00
sin signo integer 1: 0x04
sin signo integer 42: 0xa8
sin signo integer 69: 0x1501
Ejemplos de error:
~0x0100: Cero codificado en modo 1~
Opciones
Uno o cero son valores de un tipo particular. Se codifican de la siguiente forma:
0x00 si es None ("empty" o "null").
0x01 seguido del valor codificado si es Some.
Como excepción, en el caso de que el tipo sea un booleano, entonces siempre es un byte:
0x00 si es None ("empty" or "null").
0x01 si el valor es false.
0x02 si el valor es true.
Vectores (listas, series, conjuntos)
Es una colección de valores del mismo tipo codificada, que tiene un prefijo dónde se codifica con un tipo "compacto o general" el número de elementos de la colección, seguido de los elementos codificados de forma concatenada.
Tuplas y Estructuras
Una serie de valores de tamaño fijo, cada uno de ellos es posiblemente un tipo diferente de tamaño fijo. Es una concatenación de valores codificados. En las estructuras, los valores son nombrados, pero eso es irrelevante a la hora de ser codificados (los nombres son ignorados - sólo el orden importa).
Enumeraciones (tagged-unions)
Un número fijo de variables, cada una es independiente, e implica un valor adicional o una serie de valores.
El primer byte codificado índica la posición del valor de la variable. Cualquier byte adicional se utiliza para codificar el valor del dato de la variable.
Ejemplo (Rust):
enum IntOrBool {
  Int(u8),
  Bool(bool),
}
Int(42): 0x002a
Bool(true): 0x0101