# Storage

Substrate uses a simple key-value data store implemented as a database-backed, modified Merkle tree.

## Base de datos Clave-Valor (Key-Value Database)

Substrate implementa su storage (o almacenamiento) con RocksDB, almacena de manera persistentes datos en forma de clave-valor para entornos de almacenamiento rápido.

Ésto es utilizado por todos los componentes de Substrate que requieran almacenamiento persistente, tal y como son:

- Clientes de Substrate (Substrate Clients)
- Clientes "iluminados" de Substrate (Substrate light-clients)
- Trabajadores off-chain (Off-chain workers)

## Trie Abstraction ("Abstracción del árbol digital")

Una ventaja de utilizar un sistema de almacenamiento de clave-valor (key-value) es que te permite fácilmente crear estructuras abstractas de almacenamiento.

Substrate utiliza Base-16 Modified Merkle Patricia tree ("trie"), cuya implementación está en paritytech/trie, la cuál proporciona una estructura de árbol cuyo contenido puede ser modificado, y dónde el hash del root es recalculado eficientemente.

Los árboles permiten almacenan y compartir de manera eficiente el histórico de los estados de bloque. El nodo raíz del árbol o "trie root" es una representación de los datos que están dentro del árbol; es decir, dos árboles con diferentes datos siempre van a tener diferentes raices. De ese modo, dos nodos de la blockchain podrán fácilmente verificar que ellos tienen el mismo estado, simplemente comparando sus las raices del árbol.

Acceder a los datos del árbol es costoso. Cada operación de lectura requiere O(log N) de tiempo, dónde N es el número de elementos almacenados en el árbol. Para mitigar ésto, usaremos una caché de clave-valor.

All trie nodes are stored in RocksDB and part of the trie state can get pruned, i.e. a key-value pair can be deleted from the storage when it is out of pruning range for non-archive nodes. We do not use reference counting for performance reasons.

State Trie

Substrate based chains have a single main trie, called the state trie, whose root hash is placed in each block header. This is used to easily verify the state of the blockchain and provide a basis for light clients to verify proofs.
This trie only stores content for the canonical chain, not forks. There is a separate state_db layer that maintains the trie state with references counted in memory for all that is non-canonical.
Child Trie
Substrate also provides an API to generate new child tries with their own root hashes that can be used in the runtime.
Child tries are identical to the main state trie, except that a child trie's root is stored and updated as a node in the main trie instead of the block header. Since their headers are a part of the main state trie, it is still easy to verify the complete node state when it includes child tries.
Child tries are useful when you want your own independent trie with a separate root hash that you can use to verify the specific content in that trie. Subsections of a trie do not have a root-hash-like representation that satisfy these needs automatically; thus a child trie is used instead.
Runtime Storage API
The Substrate's Support module provides utilities to generate unique, deterministic keys for your runtime module storage items. These storage items are placed in the state trie and are accessible by querying the trie by key.
Next Steps
Learn More
Learn how to add storage items into your Substrate runtime modules.
Examples
View an example of creating child tries in your Substrate runtime module.
References
Visit the reference docs for paritytech/trie.