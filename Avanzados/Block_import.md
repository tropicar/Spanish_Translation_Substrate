# The Block Import Pipeline

## The Import Queue

La cola de impotación o "import queue" es una cola de trabajadores abstracta que está presente en cada nodo Substrate. No forma parte del runtime. El import queue es responsable de procesar las piezas de información entrantes, verificándolas, y si son válidas, importándolas en el estado del nodo. La información más importante que la import queue procesa son los bloques en si mismos, pero también es responsable de importar mensajes sobre los consensos, como son justificaciones, y en los light clients, pruebas de finalidad.

La import queue recolecta los elementos entrantes de las redes y los almacena en una pool. Los elementos son comprobados más tarde, para validarlos y descartarlos en el caso de que no seas válidos. Los elementos que son válidos son, entonces, importados en el estado local del nodo.

La import queue está codificada de forma abstracta en Substrate mediante el trait [ImportQueue][ImportQueue]. El uso de un trait permite que cada motor de consenso pueda proporcionar su propia implementación especializada de una import queuue, lo que puede tener ventajas de optimización como la posibilidad de verificar varios bloques en paralelo según entren en la red.

La import queue también proporciona algunos hooks en [link][link]

## The Basic Queue

Substrate proporciona una implementación en memoria predeterminada de la ImportQueue conocida como [BasicQueue][BasicQueue]. La BasicQueue no permite ningún tipo de optimización, sino que realiza los pasos de verificación e importación de manera secuencial. Sin embargo, ésto hace que la verificación sea abstracta a través del uso del trait [Verifier][Verifier].

Cualquier motor de consenso que confíe en BasicQueue debe implementar el trait de Verifier. El Verifier (Verificador) es normalmente el responsable de tareas, como la de comprobar "inherent data", y asegurar que los bloques estén firmados por la autoridad apropiada.

## The Block Import Trait

Cuando la import queue está preparada para importa un bloque, ésta pasa el bloque en cuestión, a un método proporcionado por [BasicQueue][BasicQueue]. BasicQueue no permite ningún tipo de optimización, sino que realiza los pasos de verificación e importación de manera secuencial. Substrate proporciona una implementación en memoria predeterminada de la ImportQueue conocida como Verifier trait.

Cualquier motor de consenso que confíe en BasicQueue debe implementar el trait de Verifier. El Verifier (Verificador) es normalmente el responsable de tareas, como la de comprobar "inherent data", y asegurar que los bloques estén firmados por la autoridad apropiada.

## The Block Import Pipeline

En los casos más simples, los bloque son importados directamente en el cliente. Pero la mayoría de los motores de consenso necesitarán realizar verificaciones adicionales en los bloques entrantes, actualizando sus propias bases de datos locales auxiliares, o ambas. Para permitir a los motores de consenso esta oportunidad, es común anidar el cliente en otro struct que también implemente BlockImport. Esta anidación proporciona el término "block import
pipeline".

Un ejemplo de esta forma de anidar es el [PowBlockImport][PowBlockImport],  que contiene una referencia a otro tipo que también implementa BlockImport. Esto permite al motor de consenso de PoW hacer su propio registro de importaciones, y luego pasar el bloque al BlockImport anidado, probablemente el cliente. Este patrón también se muestra en [AuraBlockImport][AuraBlockImport], en [BabeBlockImport][BabeBlockImport] y GrandpaBlockImport.

El BlockImport anidado no necesita estar limitado a un nivel. El BlockImport anidado no necesita estar limitado a un nivel. Por ejemplo, la Polkadot's block import pipeline consta de un BabeBlockImport, que envuelve un GrandpaBlockImport, que envuelve el Client.

# Aprende más

Varias de las guías de nodos explican la block import pipeline:

- [Manual_Seal][Manual Seal] todos los bloques son válidos, por lo que el block import pipeline está solo en el cliente
- [Basic_PoW][Basic PoW] el import pipeline incluye PoW y el cliente
- [Hybrid_Consensus][Hybrid Consensus] the import pipeline is PoW, then Grandpa, then the client







[ImportQueue]: https://crates.parity.io/sp_consensus/import_queue/trait
[link]: https://crates.parity.io/sp_consensus/import_queue/trait 
[BasicQueue]: https://crates.parity.io/sp_consensus/import_queue/struct
[Verifier]: https://crates.parity.io/sp_consensus/import_queue/trait
[PowBlockImport]: https://crates.parity.io/sc_consensus_pow/struct
[AuraBlockImport]: https://crates.parity.io/sc_consensus_aura/struct
[BabeBlockImport]: https://crates.parity.io/sc_finality_grandpa/struct
[Manual_Seal]: https://substrate.dev/recipes/3-entrees/manual-seal.html
[Basic_Pow]: https://substrate.dev/recipes/3-entrees/basic-pow.html
[Hybrid_Consensus]: https://substrate.dev/recipes/3-entrees/hybrid-consensus.html