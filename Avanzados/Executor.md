# Executor

El executor es el responsable de enviar y ejecutar llamadas en el runtime de Substrate.

## Runtime Execution

El runtime de Substrate se compila en un ejecutable nativo y en un binario de WebAssemblly (Wasm).
El runtime nativo forma parte del ejecutable del nodo, mientras que el binario Wasm es almacenado en la blockchain bajo una key bien conocida del storage.

Estas dos representaciones del runtime pueden no ser las mismas. Por ejemplo: cuando el runtime es actualizado. El executor determina qué versión del runtime se usa cuando se envían llamadas.

### Estrategia de ejecución

Antes de que comience a ejecutarse el runtime, el cliente de Substrate propone qué entorno de ejecución del runtime debe ser utilizado. Ésto es controlado por la estrategia de ejecución (execution strategy), que puede ser configurada para las diferentes partes del proceso de ejecución de la blockchain. Las estrategias son:

- Native: Ejecutar en nativo (si está disponible, WebAssembly); si falla, no hace uso de Wasm.

- Wasm: Solo ejecuta cuando lo compila WebAssembly.

- Both: Ejecutar ambos en nativo (cuando esté disponible) y compilarlo con WebAssembly.

- NativeElseWasm: Ejecuta con la compilación en nativo si fuera posible; si falla, entonces ejecuta con WebAssembly.

Las estrategias de ejecución en las diferentes partes del proceso de ejecución de la blockchain son:

- Syncing(Sincronización): NativeElseWasm

- Block Import(importación de bloques): NativeElseWasm

- Block Construction(Construcción de bloque): Wasm

- Off-Chain Worker: Native

- Other(otro): Native

### Wasm Execution

La representación Wasm del runtime de Substrate es considerada como el runtime canónico. Debido que el runtime de Wasm está almacenado en el storage de la blockchain, la red debe alcanzar un consenso sobre este binario. Por lo tanto, puede ser verificado para que sea consistente en todos los nodos que estén sincronizados.

El entorno de ejecución de Wasm puede ser más restrictivo que el entorno de ejecución nativo. Por ejemplo, el runtime de Wasm siempre se ejecuta en un entorno de 23-bit con una memoria configurable limitada (hasta 4 GB).

Por estas razones, la blockchain prefiere hacer la construcción de bloques con el runtime de Wasm, incluso aunque la ejecución en Wasm es considerablemente más lenta que la ejecución nativa. La misma lógica ejecutada en Wasm siempre funcionará en el entorno de ejecución nativo, pero no se puede decir lo mismo en el caso inverso. La ejecución en Wasm puede ayudar a asegurar que los productores de bloques creen bloques válidos.

### Ejecución nativa

El runtime nativo solo funcionará, solo si es utilizado por el executor, cuando ésta sea elegida como estrategia de ejecución, y sea compatible con la versión solicitada por el runtime. Para todos los otros procesos de ejecución distintos de la construcción de bloques, el runtime nativo es preferido por ser más eficiente. En cualquier situación donde el ejecutable nativo no deba ser ejecutado, el runtime canónico en Wasm, es ejecutado en su lugar.

## Actualizaciónes del Runtime

El runtime de Substrate ha sido diseñado para ser actualizado, para que puedas continuar iterando en tu función de transición de estado (Stf, State Transition Function), incluso después de que tu blockchain esté activa.

### Versiones del Runtime
Para que el executor pueda seleccionar el entorno de ejecucción del runtime apropiado, éste necesita saber el spec_name, spec_version and authoring_version de ambos, del nativo y del runtime en Wasm.

El runtime proporciona las siguientes propiedades :

- spec_name: El identificador para los diferentes runtime de Substrate.
- impl_name: El nombre de la implementación del spec.
- authoring_version: The version of the authorship interface.Un "Authoring node" no intentará escribir bloques, a menos que éste sea igual que su runtime nativo.
- spec_version: La versión de la especificación del runtime.Un nodo completo (full-node) no intentará usar su runtime nativo en sustitución del runtime en Wasm, a menos que spec_name,
spec_version, and authoring_version sean los mismos en Wasm y en nativo.
- impl_version: La versión de la implementación.Los nodos son libres de ignorar ésto; ésto sirve solo como indicativo de que el nodo es diferente; mientras que las otras dos versiones sean las mismas, entonces, el código actual puede ser diferente, a pesar de ello, es requerido que haga lo mismo.Las optimizaciones "Non-consensus-breaking" tratan sobre los únicos cambios que pueden darse como resultado de cambios en la impl_version.

Como se mencionó anteriormente, el executor siempre verifica que el runtime nativo tiene la misma lógica impulsada por consensos, antes de elegir ejecutarla, independientemente de qué versión es nueva o anterior.

Nota: Las versiones del runtime se configuran de forma manual.Por lo tanto, el executor pude todavía tomar decisiones inapropiadas si la versión del runtime está mal representada.

### Actualización del Runtime sin bifurcaciones

Las blockchaines tradicionales requieren de un [hardfork][hard fork] cuando actualizan la función de transición de estado de su cadena.Ésto requiere que los operadores de los nodos detengan sus nodos, y manualmente actualicen al ejecutable actual.Para redes distribuidas en producción, la coordinación de las actualizaciones usando hard fork puede ser un proceso complejo.

Lo que consiguen las propiedades listadas en esta página, permiten a las blockchain creadas con Substrate hacer "actualizaciones del runtime sin bifurcaciones".Ésto quiere decir que la actualización de la lógica del runtime puede ocurrir en tiempo real sin provocar ningún fork en la red.

Para llevar a cabo un actualización del runtime sin forks, Substrate utitliza la lógica del runtime existente para actualizar el runtime en Wasm almacenado en la blockchain, se actualiza a una versión nueva con la nueva lógica.Esta actualización se lleva a cabo en todos los nodos que estén sincronizados en la red como parte del proceso de consenso.Una vez el runtime en Wasm es actualizado, el executor verá que el runtime nativo spec_name, spec_version,
or authoring_version no coincide con el nuevo runtime en Wasm.Como resultado, éste volverá a ejecutar el runtime en Wasm canónico, en lugar de usar el runtime nativo para todos los procesos de ejecución.

## Siguientes Pasos

### Aprende más

- Saber más al respecto

[hardfork][https://en.wikipedia.org/wiki/Fork_(blockchain)]