# Consenso

Los nodos de la blockchain usan "consensus engines" (motores de consenso) para ponerse de acuerdo en el estado de la blockchain. Este articulo cubre las bases fundamentales del consenso en un sistema blockchain, cómo el consenso interactua con el runtime en el framework de Substrate, y el motor de consenso disponible en el framework.

## Máquinas de estado y Conflictos (State Machines and Conflicts)

Un runtime de blockchain es una [maquina de estado][maquina_de_estado]. Éste tiene algunos estados internos, y la función de transición de estados permite transicionar desde el estado actual a un estado futuro.En la mayoría de runtimes, hay estados que tiene transiciones validas a múltiples estados futuros, pero solo debe elegirse una transición.

La blockchain debe estar de acuerdo en:

- Un estado inicial, llamado "genesis".
- Series de transiciones de estado, cada una de ellas llamadas "bloque", y
- Un estado final(estado actual).

Para acordar el estado resultante después de una transición, todas las operaciones dentro de la función de transición de estado de la blockchain debe ser determinista.

## Conflicto de Exclusión

En los sistemas centralizados, la autoridad central elige entre los alternativas exclusivas registrando las transiciones de estado en el orden en que las ve, cuando aparece un conflicto, elige la primera de las alternativas que están compitiendo. En sistemas descentralizados, los nodos verán las transacciones en orden diferente, y por tanto, ellos deben elaborar métodos para excluir transacciones. Como complicación adicional, las redes blockchain se esfuerzan para ser tolerantes con los fallos, lo que quiere decir que ellas deber continuar proporcionando datos consistentes, incluso si alguno de los participantes no sigue las reglas.

Las blockchains ponen las transacciones en bloques y disponen de algún método para seleccionar qué participante tiene el derecho para subir/enviar un bloque. Por ejemplo, en una blockchain de Proof-Of-Work, el nodo que encuentre una proof of work válido primero, tiene el derecho para subir un bloque a la cadena.

Substrate proporciona varios algoritmos de construcción de bloque, y también permite crear el tuyo propio:

- Aura (round robin)
- BABE (slot-based)
- Proof of Work

## Reglas de bifurcación

De forma simple, un bloque contiene una cabecera y un conjunto de "extrinsics". El header(cabecera) debe contener una referencia de su bloque padre para que pueda ser traceable hasta su génesis. Las bifurcaciones o Forks ocurren cuando 2 bloques tienen como referencia un mismo padre. Los Forks deben ser resueltos de tal forma que solo exista una cadena canónica.

Una regla de bifurcación es un algoritmo que toma una blockchain y selecciona la "mejor" cadena, por tanto, esa debe ser la que continue extendiéndose.Substrate expone este concepto a través del
[SelectChain][SelectChain].

Substrate permite escribir una regla personalizada de elección de bifurcación, o usar una que "salga de la caja". Por ejemplo:

## Regla de la Cadena más Larga

Substrate expone esta regla de selección de la cadena más larga en [LongestChain][LongestChain]. GRANDPA utiliza la regla de cadena más larga para votar.

## GHOST Rule (Regla GHOST)

La regla "The Greedy Heaviest Observed SubTree" dice que, empezando en el bloque génesis, cada fork o bifurcación se resuelve eligiendo la rama que tiene más bloques creados de forma recursiva.

# Producción de Bloques

Algunos nodos en una red blockchain son capaces de crear nuevos bloques, un proceso conocido como "authoring". Los nodos que pueden crear firmar bloques depende del "consensus engine" que se esté utilizando. En una red centralizada, un único nodo puede crear todos los bloques, mientras que en una red sin ningún mensaje, un algoritmo debe seleccionar el creador de cada bloque.

## Proof Of Work

En Proof-Of-Work, sistemas como Bitcoin, cualquier nodo puede producir un bloque en cualquier momento, siempre que haya resuelto un problema de computación. La resolución de problemas requiere de tiempo de CPU, y por tanto, los mineros pueden solo producir bloques en proporción a sus recursos de computación. Substrate dispone de un motor de producción de bloques usando proof of work.

## Slots

Los algoritmos de consenso basados en Slot (Slot-based) deben tener un conjunto de validadores conocidos a quienes se les permita producir bloques. El tiempo se divide en pequeños slots, y en cada slot, algunos pueden producir un bloque. Las especificaciones de qué validadores pueden crear bloques durante cada slot difieren entre los diferentes engines. Substrate proporciona Aura y Babe, ambos son motores de creación de bloques basados en slots (slot-based block authoring engines).

# Finalidad

Substrate proporciona Aura y Babe, ambos son motores de creación de bloques basados en slots (slot-based block authoring engines). En algunos sistemas tradicionales, el fin llega cuando un recibo es entregado, o los papers han sido firmados. Utilizando los esquemas de creación de bloques y las reglas de selección de bifurcaciones que se han discreto, las transacciones nunca son completamente finalizadas. Siempre hay una oportunidad de que haya una cadena más larga (o más pesada) aparezca y revierta tu transacción. Sin embargo, cuando más bloques se construyan sobre un bloque en particular, menos probable es que éste sea revertido. De esta manera, la creación de bloque junto con una regla de bifurcación adecuada proporciona un final probabilístico. 

Cuando se desea una finalidad determinista, se puede añadir un gadget a la lógica de la blockchain. Los miembros de un conjunto de la authority cuentan los votos, y cuando un bloque tiene suficientes votos, este bloque es considerado el final. En la mayoría de los sistemas, este umbral es 2/3. Los bloques que han sido acabados por este gadget no pueden set revertidos sin una coordinación externa como un "hard fork".

Algunos sistemas de consenso combinan la producción de bloque y la finalidad, de la siguiente forma, la finalización es parte del proceso de producción de bloques, y un nuevo bloque N+1 no puede ser firmada hasta que el bloque N acabe. Substrate, sin embargo, aisla los dos procesos y te permite usar cualquier motor de producción de bloques por si solo, con una finalidad probabilística o usarlo junto con un gadget de finalidad para tener finalidad determinista.

En los sistemas que utilizan un gagdet de finalidad, la regla de elección de bifurcación (fork choice rule) debe ser modificada para considerar los resultados del "finality game". Por ejemplo, en lugar de tomar la cadena más larga, un nodo podría seleccionar la cadena más larga pero que contenga el bloque finalizado más reciente.

# Consenso en Substrate

El framework de Substrate viene con varios motores de consenso que permiten la creación de bloques, o l
a finalidad. Este artículo proporciona una breve visión general de las ofertas incluidas en Substrate. Este artículo proporciona una breve visión general de las ofertas incluidas en Substrate.

## Aura

[Aura][Aura] proporciona un mecanismo de creación de bloques basados en slot (slot-based block authoring mechanism). En Aura un conjunto conocido de autoridades se turnan a la hora de producir los bloques.

## BABE

[BABE][BABE] también proporciona la creación de bloques basadas en slot(slot based block authoring) con un conjunto de validadores conocidos. De esta manera es similar a Aura. A diferencia de Aura, la asignación de slot está basada en la evaluación de una función aleatoria verificable (VRF, Verifiable Random Function). Cada validador tiene asignado un peso para un epoch. Este "epoch" está compuesto por slots, y el validador evalúa su VRF en cada slot. Para cada slot en la que la salida de la VRF del validador esté por debajo de su peso, se le permitiría crear un bloque.

Debido a que varios validadores pueden producir un bloque en el mismo slot, los forks o bifurcaciones son más comunes en BABE que en Aura, y es, incluso, común en una red en buenas condiciones.

La implementación de Substrate de BABE también tiene un mecanismos de reserva (fallback mechanism) en el caso de que ninguna autoridad haya sido elegida en un slot.Estas asignaciones "secundarias" de un slot permiten a BABE lograr un tiempo de bloque constante.

## Proof Of Work

[proof of work][proof of work] no está basada en slots, y no requiere de un conocido conjunto de autoridad. En proof o work, cualquiera puede crear un bloque en cualquier cualquier momento, siempre y cuando resuelva un problema computacional complicado (normalmente, es la búsqueda de un "hash preimage"). La dificultad de este problema puede ser ajustada para proporcionar un tiempo de bloque estático.

## GRANDPA

[GRANDPA][GRANDPA] proporciona finalización de bloque. Tiene una conocida autoridad ponderada como BABE. Sin embargo, GRANDPA no crea bloques; solo escucha "gossip" (literalmente, chismes o cotilleos, se refiere a información en este caso) sobre los bloques que han sido producidos por algún motor de creación de bloques como los 3 que hemos mencionado anteriormente. Los validadores de GRANDPA votan sobre cadenas, no sobre los bloques,por ejemplo, ellos votan un bloque que consideran "el mejor" y sus votos se aplican de forma transitoria a todos los bloques anteriores. Una vez más de los 2/3 de las autoridades de GRANDPA hayan votado por un bloque en particular, éste se le considera el final.

## Coordinación con el Runtime

Los algoritmos de consenso estáticos más simples funcionan desde fuera del runtime como se ha descrito anteriormente. Sin embargo, muchos consensos son más potentes si se añaden características que requieran de la coordinación con el runtime. Los ejemplos incluyen una dificultad ajustable en proof of work, rotación de la autoridad en proof of authority, y la ponderación en pesos en las redes de proof of stake.

Para incluir estascaracterísticas de consenso, Substrate dispone del concepto de DigestItem, un mensaje que va desde la parte externa del nodo, dónde se hace el consenso, al runtime, o viceversa.

# Aprende más

Debido a que, tanto BABE como GRANDPA se utilizarán en la red de Polkadot, Web3 Foundation proporciona presentaciones a nivel de investigación de los algoritmos.

- Investigación BABE
- Investigación de GRANDPA

Todos los algoritmos de finalidad determinísticos, incluyendo GRANDPA, requieren al menos 2f + 1 nodos no defectuosos
, donde f es el número de nodos defectuosos o maliciosos. Aprenda más sobre de donde viene este umbral, y por qué es el ideal en un paper https://lamport.azurewebsites.net/pubs/reaching.pdf" Reaching Agreement in the Presence of Faults o en https://en.wikipedia.org/wiki/Byzantine_fault Wikipedia: Byzantine Fault.

No todos los protocolos de consenso definen una cadena única y canónica. Algunos protocolos validan https://en.wikipedia.org/wiki/Directed_acyclic_graph" directed acyclic graphs (DAG, grafos acíclicos dirigidos) cuando dos bloques con el mismo padre no tienen cambios de estado de conflicto.











maquina_de_estado: https://en.wikipedia.org/wiki/Finite-state_machine
SelectChain: https://crates.parity.io/sp_consensus/trait
LongestChain: https://crates.parity.io/sc_consensus/struct
Aura: https://crates.parity.io/sc_consensus_aura/index.html
BABE: https://crates.parity.io/sc_consensus_babe/index.html
proof of work: https://crates.parity.io/sc_consensus_pow/index.html
GRANDPA: https://crates.parity.io/sc_finality_grandpa/index.html