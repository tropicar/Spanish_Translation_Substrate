# Introducción

Substrate es una plataforma de Blockchain con una Función de Transición de Estado (STF, State Transition Function) genérica y componentes modulares para consensos, redes y configuración.
A pesar de ser "completamente genérico", dispone de estándares y convencionalismos - especialmente, con el Substrate runtime module library (También conocido como FRAME) - con respecto a las estructuras de datos que recibe el STF (State Transition Function), hace realidad el rápido desarrollo de blockchain.

## Tipos de datos básicos

Hay varios tipos de datos que funcionan bajo el núcleo de Substrate (éstos son conocidos como "Core data types"). Es obligatorio el definirlos y deben tener una interfaz particular para poder trabajar con el framework Substrate.

Cada uno de estos tipos de datos se corresponde en Rust a lo que se conoce como trait. Éstos son:
Hash, un tipo que codifica un resumen criptográfico de algunos datos. Normalmente solo 256 bits.
BlockNumber, un tipo que codifica el número total de antepasados que tenga cualquier bloque válido. Normalmente 32 bits de tamaño.

DigestItem, un tipo que debe ser capaz de codificar una de alternativas fijas (éstas no pueden ser modificadas) para el consenso y el seguimiento de cambios, así como variantes programadas, relevantes para módulos específicos dentro del "runtime".

Digest, básicamente solo una serie de DigestItems, ésto codifica toda la información que es relevante para que un ""cliente ligero" tenga a mano dentro del bloque.

Header, un tipo que es representativo (criptográfico o no) de toda la información relevante de un bloque. Ésta incluye el hash del padre, la raíz de almacenamiento, la raíz del árbol de "extrinsics", el "digest" y un número de bloque.

Extrinsic, un tipo para representar una sola pieza de datos externa a la Blockchain que es reconocida por la Blockchain. Ésto típicamente involucra una o más firmas, y algún tipo de instrucción codificada (por ejemplo, para transferir la propiedad de los fondos o llamar a un smart contract).

Block, esencialmente es solo una combinación del Header y una serie de Extrinsics, junto con una especificación sobre el algoritmo que se ha utilizado para para obtener el hash.

Las referencias de las implementaciones genéricas de cada uno de estos "trait" están disponibles en la SRML(Substrate Runtime Module Library). Técnicamente no son necesaria utilizarlas, pero hay algunos casos dónde pueden no ser lo suficientemente genéricas para un caso práctico.

Se necesita algo de experiencia
Con el fin de sacar el mayor provecho de Substrate, se debería tener un buen conocimiento sobre los conceptos de Blockchain y de criptografía básica. Terminología como cabecera(header), bloque(block), client(cliente), hash, transacciones(transaction) y firmas(signature) deberían serte familiar. Por el momento necesitarás tener un conocimiento funcional de Rust para poder ser capaz de realizar pequeñas personalizaciones/adaptaciones de Substrate (aunque normalmente, nuestro objetivo es que ese no sea el caso).

## Uso

Substrate está diseñado para ser utilizado de alguna de las 3 formas siguientes:

- Con nuestro nodo incluido(bundle Node): ejecutando el nodo prediseñado de Substrate (substrate), Substrate Node, y configurándolo con un bloque génesis, que incluye el actual runtime de demostración. En este caso, simplemente, configure un archivo JSON y ponga en marcha su propia Blockchain. Ésto permite muy poca personalización, solamente se podrían modificar parámetros del génesis, los cuáles están incluidos en el módulo runtime node, estos parámetros serían; balances, staking, periodo de bloques, fees y governance. Tienes un tutorial utilizando Substrate de esta manera, aquí Deploying a Substrate Node chain.

- Con SRML(Substrate Runtime Module Library, o FRAME): Haciéndo uso de SRML para crear un nuevo "runtime", ya sea añadiendo nuevos módulos personalizados, personalizándolos, o reconfigurando la lógica de creación de bloques del cliente. Ésto nos concede mucha libertad en la lógica de nuestra blockchain, nos permite modificar datatypes (tipos de datos), seleccionar la librería de módulos, e incluso, añadir tus propios módulos. Se pueden realizar numerosas modificaciones sin tocar la lógica de creación de bloques (block-authoring logic), ya que, ésta es controlada por la lógica "on chain" (on-chain logic). Si éste fuera el caso, entonces, el binario ya existente de Substrate puede ser utilizado para la creación y sincronización de bloques. Si la lógica de creación de bloques necesita ser retocada, entonces, se debe crear un nuevo binario con la lógica de creación de bloques en un proyecto separado, y que, será utilizado por los validadores. Así es cómo la "relay chain" de Polkadot está construida, y debería satisfacer casi todas las circunstancias a medio plazo. Para realizar un tutorial sobre esta forma de trabajar, dirigite a creating your first Substrate chain.

- Genérico (Substrate Core): Todo el SRML puede ser ignorado, de tal forma que, podrías diseñar el "runtime" e implementarlo desde cero. Si lo desea, podría hacerlo en cualquier otro lenguaje que no fuese Rust, siempre y cuando, pudiera comunicarse con WebAssembly. Si el "runtime" fuera creado, de forma que, fuera compatible con la lógica abstracta de la creación de bloques del Substrate Node, entonces, podrías simplemente, crear un nuevo bloque génesis de tu "wasm blob" y poner en marcha tu blockchain con el cliente Rust-based Substrate. Si no, pues, necesitarás modificar la lógica de creación de bloques del cliente, y puede que incluso también requiera modificar el header y el formato de serialización de bloques. En términos de comodidad de desarrollo, ésta es, por mucho la forma más complicada de utilizar Substrate, aunque es la que más te permite innovar. Permitirá una ruta de actualización a largo plazo en el padarigma de Substrate.