# Storage

Substrate uses a simple key-value data store implemented as a database-backed, modified Merkle tree.

## Base de datos Clave-Valor (Key-Value Database)

Substrate implementa la base de datso del storage con [RocksDB][RocksDB], que es una base de datos del tipo clave-valor para entornos de almacenamiento rápido. También tiene soporta para un sistema de almacenamiento experimental llamado [ParityDB][ParityDB].

Ésto es utilizado por todos los componentes de Substrate que requieran almacenamiento persistente, tal y como son:

- Clientes de Substrate (Substrate Clients)
- Clientes "ligeros" de Substrate (Substrate light-clients)
- Trabajadores off-chain (Off-chain workers)

## Trie Abstraction ("Abstracción del árbol digital")

Una ventaja de utilizar un sistema de almacenamiento de clave-valor (key-value) es que permite fácilmente crear estructuras abstractas de almacenamiento.

Substrate utiliza Base-16 Modified Merkle Patricia tree ("trie"), cuya implementación está en [paritytech/trie][paritytech/trie], la cuál proporciona una estructura de árbol cuyo contenido puede ser modificado, y dónde el hash del root es recalculado eficientemente.

Los árboles permiten almacenan y compartir de manera eficiente el histórico de los estados de bloque. El nodo raíz del árbol o "trie root" es una representación de los datos que están dentro del árbol; es decir, dos árboles con diferentes datos siempre van a tener diferentes raices. De ese modo, dos nodos de la blockchain podrán fácilmente verificar que ellos tienen el mismo estado, simplemente comparando sus las raices del árbol.

Acceder a los datos del árbol es costoso. Cada operación de lectura requiere O(log N) de tiempo, dónde N es el número de elementos almacenados en el árbol. Para mitigar ésto, usaremos una caché de clave-valor.

Todos los nodos del árbol están almacenados en la DB (database, Base de datos), y parte del estado del árbol puede ser podada, es decir, un par clave-valor puede ser borrada del storage cuando éstos están fuera del rango de "purga" o de "poda (pruning range)" para "non-archive nodes".No utilizamos [reference counting][reference counting] por problemas de rendimiento.

### State Trie 

Las blockchain basadas en Substrate tiene un solo árbol, llamado "state trie"(árbol de estado),  en el que el hash del root está colocado en la cabecera de cada bloque.Ésto se utiliza para verificar fácilmente el estado de la blockchain y proporcionar una base a los los "light clients" (clientes ligeros) para verificar ""proofs".

Hay una capa separada llamada [state_db][state_db] que se encarga de mantener el "trie state" con referencias contadas en memoria para todo lo que no es de la cadena principal.

### Child Trie

Substrate también proporciona una API para generar nuevos "child tries" con sus propios hashes de la raíz que pueden ser utilizados en el runtime.

Los child tries son idénticos que el state trie principal, salvo en que la raíz del child trie se almacena y se actualiza como si fuera un nodo en el árbol principal, en lugar de, en la cabecera del bloque.Dado que sus cabeceras son parte del state trie principal, es fácil verificar completamente el estado del nodo cuando éste incluye child tries.

Los child tries son útiles cuando quieres tu propio árbol independiente con un hash de la raíz separado, para así poder verificar contenido específico en dicho árbol. Las subsecciones de un árbol no tiene una forma de hacer a pequeña escala lo que hace la raíz del árbol de manera automática; por lo tanto, se utilizan child trie.

## Consultas al Storage

Las Blockchains que son creadas con Substrate tienen un servidor RPC(Remote Procedure Call) que puede ser utilizado para hacer consultar al runtime del storage. Cuando usas el RPC de Substrate para acceder a un item del storage, solo necesitas la clave asociada a ese item.Substrate's runtime storage APIs disponen de un número de tipos de item de storage; continua leyendo para aprender como calcular claves de storage para los diferentes tipos de items de storage.

### Storage value keys

Para calcular la clave para un valor simple del storage, calcula el TwoX 128 hash del nombre del módulo que contiene el valor del storage y añádelo al hash TwoX 128 del nombre del valor del Storage. Por ejemplo, el pallet [Sudo][Sudo] dispone de un item llamado Module.html#method.key">Key:

~~~
twox_128("Sudo")                   = "0x5c0d1176a568c1f92944340dbfed9e9c"
twox_128("Key)                     = "0x530ebca703c85910e7164cb7d1c9e47b"
twox_128("Sudo") + twox_128("Key") = "0x5c0d1176a568c1f92944340dbfed9e9c530ebca703c85910e7164cb7d1c9e47b"
~~~

Si la conocida cuenta de Alide es el "sudo user", una petición RPC y responde para leer el Sudo module's key, el Storage Value podría ser representado como: 

~~~
state_getStorage("0x5c0d1176a568c1f92944340dbfed9e9c530ebca703c85910e7164cb7d1c9e47b") = "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"
~~~

En ese caso, el valor que devuelve (0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d) es el ID(5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY) de la cuenta de Alice codificado en SCALE.

Puede que hayas notado que el algoritmo de hash TwoX 128 no es criptográfico y que se utiliza para generar las Storage Value keys. Ésto se debe a que no es necesario pagar el coste del rendimiento asociado a una función criptográfica de hash dado que la entrada de la función del hash( los nombres del módulo y el storage item) está determinada por el desarrollador del runtime y no por los posibles usuarios maliciosos de la blockchain.

### Storage Map Keys

Al igual que los Storage values, las claves de Storage Maps  se calcula el hash con la función TwoX 128 del nombre del módulo que contiene el mapa seguido del hash del nombre del propio Storage Map. Para recuperar un elemento del mapa, simplemente añade el hash de la key del mapa que desees a la storage key del Storage Map.Para mapas que tengan 2 claves (Storage Double Maps), añade el hash de la clave del primer mapa seguida del hash de la clave del segundo mapa a la Storage Double Map's storage key.Al igual que las Storage values, Substrate usará el algoritmo de hash Two 128 en el nombre del módulo y en el nombre de Storage Map, pero necesitarás asegurarte de usar el algoritmo correcto (El que fue declarado en el macro decl_storage) cuando determines los hashes de las claves para los elementos en el mapa.


## Referencias

[RocksDB]: https://rocksdb.org/
[ParityDB]: https://github.com/paritytech/parity-db
[paritytech/trie]: https://github.com/paritytech/trie
[reference counting]: http://en.wikipedia.org/wiki/Reference_counting
[state_db]: https://crates.parity.io/sc_state_db/index.html
[Sudo]: https://crates.parity.io/pallet_sudo/index.html

RocksDB: https://rocksdb.org/ 

ParityDB: https://github.com/paritytech/parity-db

paritytech/trie: https://github.com/paritytech/trie

reference counting: http://en.wikipedia.org/wiki/Reference_counting

state_db: https://crates.parity.io/sc_state_db/index.html

Sudo: https://crates.parity.io/pallet_sudo/index.html