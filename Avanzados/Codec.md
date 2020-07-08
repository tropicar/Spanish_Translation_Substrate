# Codificación SCALE

La codificación SCALE (Simple Concatenated Aggregate Little-Endian) es una codificación ligera, eficiente, de serialización y deserialización binaria.

Ha sido diseñado para alto rendimiento, codificación y decodificación copy-free de los datos en entornos con limitaciones de recursos, como el runtime de Substrate. 

## SCALE en Substrate

Substrate utiliza el [parity-scale-codec][parity-scale-codec], una implementación en Rust para codificar en SCALE. Esta librería y la codificación SCALE son ventajosas para Substrate y los sistemas blockchains porque:

- Es ligero en comparación a los frameworks genéricos de serialización como serde, que añaden código que no es necesario y que puede inflar el tamaño del binario.
- No utiliza Rust STD, y por tanto, puede compilar Wasm para el runtime de Substrate.
- Esta creado para funcionar con Rust, y derivar la lógica de codificación para nuevos tipos:

~~~
 #[derive(Encode, Decode)]
~~~

Es importante definir el esquema de codificación utilizado en Substratem en lugar de reutilizar una librería de codificación de Rust, porque este sistema de codificación necesita ser reimplementado en otras plataformas y lenguajes.

## Definición de la Codificación (Codec Definition)

Aquí encontrara como es la codificación SCALE codifica los diferentes tipos.

### Enteros de tamaño fijo

Los enteros básicos (Basic Integers) son codificados usando el formato tamaño fijo little-Endian(LE).

#### Ejemplo

- signed 8-bit integer 69: 0x45

- unsigned 16-bit integer 42: 0x2a00

- unsigned 32-bit integer 16777215: 0xffffff00

### Enteros compactos/generales (Compact/General Integers)

Una codificación de enteros "compactos" o general (compact or general Integer) es suficiente para codificar enteros de gran tamaño (large integer, mayores hasta 2\*\*536) y es más eficiente a la hora de codificar la mayoría de valores en comparación a la versión de tamaño fijo. (Aunque para valores de un solo byte, syngle-byte values, el entero de tamaño fijo nunca es peor)

Se codifica con los 2 bits menos significativos, los cuáles indican el modo:

- 0b00: single-byte mode; los seis bits superiores son el valor codificados en Little Endian(válido solo para valores comprendidos entre 0-63).

- 0b01: two-byte mode: los seis bits superiores y el byte siguiente corresponden al valor codificado en LE (solo válido para valores entre 64-(2**14-1)).

- 0b10: four-byte mode: los seis bits superiores y los siguientes tres bytes son el valor codificado en LE (solo válido para valores entre (2**14-1)-(2**30-1)).

- 0b11: Big-integer mode: Los seis bits superiores son el número de bytes, menos cuatro. El valor está contenido, codificado en LE, en los bytes que se han definido menos 4. El byte final (el más significativo) debe ser distinto de cero. Válido solo para valores entre ((2**30-1)-(2**536-1)).

#### Ejemplo

- unsigned integer 0: 0x00

- unsigned integer 1: 0x04

- unsigned integer 42: 0xa8

- unsigned integer 69: 0x1501

Error:

- ~~0x0100: Zero encoded in mode 1~~

### Boolean

Los valores de tipo Boolean son codificados usando el bit menos significativos de un solo byte.

#### Ejemplo

- boolean false: 0x00

- boolean true: 0x01

### Opciones

Valores uno o cero de un tipo particular. Codificado como:

- 0x00 Si éste es None ("empty" or "null").

- 0x01 seguido por el valor codificado si éste es Some.

Como excepción, en el caso de que el tipo sea un Boolean, éste siempre es un byte:

- 0x00 Si éste es None ("empty" or "null").

- 0x01 si este valor es false.

- 0x02 si este valor es true.

### Resultados

Los resultados suelen ser enumeraciones que indican si ciertas operaciones fueron realizadas con éxito o fallidas. Codificado como:

- 0x00 Si la operación es exitosa, seguida del valor codificado.

- 0x01 si la operación fuera fallida, seguida del error codificado.

#### Ejemplo

~~~
// Un tipo resultado personalizado usado en un "crate".
let Result = std::result::Result<u8, bool>;
~~~

- Ok(42): 0x002a

- Err(false): 0x0100

### Vectores (listas, series, conjuntos)

Para codificar una colección de valores del mismo tipo, se utiliza un prefijo de codificación id="4">compact (compact) del número de items, seguido de los los items codificados uno detrás de otro.

#### Ejemplo

Vector de unsigned 16-bit integers (enteros sin signo de 16-bit):

[4, 8, 15, 16, 23, 42]

ESCALE Bytes:

0x18040008000f00100017002a00

### Strings

Strings son Vectores que contienen una secuencia válida en UTF8.

### Tuplas (Tuples)

Una serie de valores de tamaño fijo, cada uno es posiblemente un tipo diferente pero predeterminado y de tipo fijo. Ésto es simplemente la concatenación de cada valor codificado.

#### Ejemplo

Tupla de un entero compacto sin signo y un boolean:

~~~
(3, false): 0x0c00
~~~

### Data Structures

Para estructuras, los valores son nombrados, pero ésto es irrelevante a la hora de codificar (los nombres son ignorados, lo único que importa es el orden). Todos los contenedores almacenan elementos de forma consecutiva. El orden de los elementos no es fijo, depende del contenedor, y no del decodificador.

Ésto implícitamente quiere decir que decodificar algún array de bytes en una estructura que impone un orden, y luego recodificarlo, podría convertir el orifinal en un array de bytes diferente al que se codificó en primer lugar.

#### Ejemplo

Imagina una estructura SortedVecAsc que siempre tiene los byte de un elemento en orden ascencente, y tienes [3, 5, 2, 8], dónde el primer elemento es el número de bytes siguiente (por ejemplo [3, 5, 2] sería inválido).

SortedVecAsc::from([3, 5, 2, 8]) decodificaría [3, 2, 5, 8], el cuál no concuerda con la codificación original.

### Enumerados (tagged-unions)

Un número fijo de variables, cada una es independiente, e implica que es muy posible que haya un valor adicional o una serie de valores.

El primer byte codificado identifica la posición del valor de la variable. Cualquier byte adicional se utiliza para codificar cualquier dato relacionado con la variable. Por lo tanto, no se admiten más de 256 variables.

#### Ejemplo

~~~
enum IntOrBool {
  Int(u8),
  Bool(bool),
}
~~~
- Int(42): 0x002a

- Bool(true): 0x0101

## Implementaciones

La codificación SCALE de Parity ha sido implementada en muchos lenguajes, incluyendo una implementación escrita en Rust y mantenida por Parity Technologies.

- Rust: [paritytech/parity-scale-codec][paritytech/parity-scale-codec]
- Python: [polkascan/py-scale-][polkascan/py-scale-]
- Golang: [ChainSafe/gossamer][ChainSafe/gossamer]
- C++: [soramitsu/scale][soramitsu/scale]
- JavaScript: [polkadot-js/api][polkadot-js/api]
- AssemblyScript: [LimeChain/as-scale-codec][LimeChain/as-scale-codec]
- Haskell: [airalab/hs-web3][airalab/hs-web3]
- Java: [emeraldpay/polkaj][emeraldpay/polkaj]
- Ruby: [itering/scale.rb][itering/scale.rb]

## Referencias

- Visita los documentos de referencia para la [paritytech/parity-scale-codec][paritytech/parity-scale-codec].
- Visite la sección de codificación auxiliar de la [especificación-del-entorno-del-runtime-de Polkadot][especificación-del-entorno-del-runtime-de Polkadot].

[parity-scale-scoded]: https://github.com/paritytech/parity-scale-codec
[paritytech/parity-scale-codec]: https://github.com/paritytech/parity-scale-codec
[polkascan/py-scale-]: https://github.com/polkascan/py-scale-codec
[ChainSafe/gossamer]: https://github.com/ChainSafe/gossamer
[soramitsu/scale]: https://github.com/soramitsu/kagome/tree/master/core/scale
[polkadot-js/api]: https://github.com/polkadot-js/api
[LimeChain/as-scale-codec]: https://github.com/LimeChain/as-scale-codec
[airalab/hs-web3]: https://github.com/airalab/hs-web3/tree/master/src/Codec
[emeraldpay/polkaj]: https://github.com/emeraldpay/polkaj
[itering/scale.rb]: https://github.com/itering/scale.rb
[especificación-del-entorno-del-runtime-de Polkadot]: https://github.com/w3f/polkadot-spec/
