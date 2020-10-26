# Información Background

En esta sección vamos a enseñarle acerca el framework de desarrollo de blockchain Substrate.

## Blackground en Blockchains

El desarrollo Blockchain es difícil.

Las redes blockchain se componen de nodos individuales conectados entre si en una red peer-to-peer (P2P). Los nodos son ordenadores individuales en una red donde se está ejecutando el software de blockchain, el cuál hace que todo funcione.

Para funcionar, un nodo blockchain necesita:
- Una Base de datos
- Red P2P
- Algoritmo de consenso (A consensus Engine)
- Manejo de transacciones (Transaction Handling)
- Manejo de transacciones (Transaction Handling)
- y más...

Estas tecnologías abarcan muchos campos dentro de la informática y, por lo tanto, requiere de equipos de expertos para desarrollarse. Debido a ello, la mayoría de proyectos de Blockchain no son desarrollados desde cero. En lugar de eso, estos proyectos están bifurcados desde repositorios de blockchain ya existentes. Por ejemplo:
El repositorio de Bitcoin fue bifurcado para crear: Litecoin, ZCash, Namecoin, Bitcoin Cash, etc...
El repositorio de Ethereum fue bifurcado para crear: Quorem, POA Network, KodakCoin, Musicoin, etc...

Crear blockchains de esta manera tiene serias limitaciones, ya que, estas plataformas no fueron diseñadas para que fuesen modificadas.

## Substrate

Substrate es un framework de código abierto, modular y extensible para crear blockchains.

Substrate fue diseñado desde cero para proporcionar un framework flexible, donde innovadores pudiesen diseñar y crear sus próxima red de blockchain. Ésta dispone de todos los componentes que se necesitan para crear un nodo de blockchain personalizado.

### Plantilla de nodos Substrate

Proporcionamos una un nodo hecho en Substrate, en forma de plantilla,que debería estar compilando mientras lees ésto. Ésta dispone de todos los componentes que se necesitan para crear un nodo de blockchain personalizado!

Os enseñaremos como utilizar este nodo en modo "desarrollo", el cuál permite ejecutar una red con un solo nodo, y tener cuentas de usuario preconfiguradas con fondos.

### Substrate Pallet Template

El framework Substrate (FRAME) pone énfasis en facilitar el el diseño personalizado de la lógica de ejecución de bloques (custom block execution logic), también conocido como la función de transición de estado (State Transaction Function) de la blockchain. Llamamos a esta parte de substrate el runtime.

El runtime de Substrate está compuesto por pallets de FRAME. ¡Puedes pensar en estos pallets como piezas lógica individuales que definen lo que tu blockchain puede hacer! Substrate le proporciona un número de pallets preconstruidos con el framework FRAME.

Por ejemplo, FRAME incluye una pallet de Balances que controla moneda de tu blockchain gestionando el balance de todas las cuentas en tu sistema.

Si quiere añadir la funcionalidad del smart contract a su blockchain, solo necesitas incluir el pallet de Contracts.

Incluso cosas como la governanza on-chain puede ser añadida a tu blockchain incluyendo pallets como Democracy,Elections y Collective.

El objetivo de este tutorial es enseñarte cómo crear tu propio pallet Substrate que será incluido en tu blockchain personalizada! La plantilla del nodo Substrate substrate-node-template viene con un pallet de plantilla con el que crearemos la lógica que queramos.

## Proof Of Existence Chain

La lógica personalizada que añadiremos a su Substrate runtime es un pallet de "Proof of Existence". De [Wikipedia][Wikipedia]:

Proof of Existence is an online service that verifies the existence of computer files as of a specific time via timestamped transactions in the bitcoin blockchain.

En lugar de subir todo el archivo para "demostrar su existencia", los usuarios suben hash del archivo, conocido como un "file digest" (resumen) o checksum. Estos digests o resúmenes son una importante utilidad ya que enormes archivos pueden ser representados con un pequeño valor de un hash, lo cuál es eficiente a la hora de almacenar documentos en la blockchain. Cualquier usuario con un el archivo original puede comprobar que el archivo coincide con el que se encuentra en la blockchain, simplemente recalculando el hash del archivo y comparándolo con el hash almacenado en la blockchain.

Para añadir a ésto, los sistemas blockchain también proporcionan un sistema de cuentas robusto. Así que cuando un "digest" o resumen de un archivo está almacenado en la blockchain, podemos también grabar qué usuario subió ese resumen. Ello permite a este usuario demostrar más tarde que es la persona original para reclamar el archivo.

Nuestro pallet de "Proof of Existence" expondrá 2 funciones que pueden ser llamadas:
create_claim - permite a los usuarios reclamar la existencia de un archivo subiendo un resumen o "file digest" de éste.

revoke_claim - permite al actual dueño de una reclamación revocar su propiedad.
Solo necesitaremos almacenar información sobre los "proofs" (pruebas) que han sido reclamadas, y quienes han realizado dichas reclamaciones.

Suena bastante siemple ¿no?

Si su nodo ha acabado de compilar, ya está preparado para la siguiente sección, dónde interactuaremos con la plantilla del nodod Substrate (Substrate Node Template)!

[wikipedia]:https://en.wikipedia.org/wiki/Proof_of_Existence