# Build

Ahora vamos a modificar el substrate-node-template para presentar la funcionalidad básica de un pallet de "Proof Of Existence".
Abre el substrate-node-template en tu editor de código favorito. Entonces, abre el archivo runtime/src/template.rs

~~~
substrate-node-template
|
+-- runtime
|   |
|   +-- Cargo.toml
|   |
|   +-- build.rs
|   |
|   +-- src
|       |
|       +-- lib.rs
|       |
|       +-- template.rs  <-- Edita este archivo
|
+-- scripts
|
+-- src
|...
~~~

Verás un código por defecto que usaremos como plantilla para un nuevo pallet. Puedes eliminar el contenido de este archivo, ya que, vamos a escribir uno desde cero para que sea transparente todo el proceso.

## Crea tu nuevo Pallet

A alto nivel, un pallet de Substrate puede dividirse en cinco secciones:

- 1. Imports (importar librerías)
`use frame_support::{decl_module, decl_storage, decl_event, dispatch};`
`use system::ensure_signed;`

- 2. Pallet Configuration (configuración del pallet)
`pub trait Trait: system::Trait { /* --snip-- */ }`

- 3. Pallet Events (eventos en el pallet)
`decl_event! { /* --snip-- */ }`

- 4. Pallet Storage Items (Almacenamiento de items del Pallet)
`decl_storage! { /* --snip-- */ }`

- 5. Callable Pallet Functions (funciones del pallet que pueden ser llamadas)
`decl_module! { /* --snip-- */ }`

Los events (eventos), storage (almacenamiento), y callable functions (funcionas llamables) deberían serte familiar en el caso de que hayas trabajado en el desarrollo blockchain. Te mostraremos cómo se ven cada uno de estos componentes para un pallet básico de "Proof Of Existence".

### Importaciones

Ya que las importaciones son bastante aburridas, puedes empezar copiando esto en la parte superior de tu archivo template.rs:

`use frame_support::{decl_module, decl_storage, decl_event, dispatch, ensure, StorageMap};`
`use system::ensure_signed;`
`use sp_std::vec::Vec;`

### Configuración del Pallet
Por ahora, lo único que vamos a configurar sobre el pallet es que emitirá algunos eventos.

~~~
/// La configuración del trait del pallet.
pub trait Trait: system::Trait {
    /// The overarching event type.
    type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
}
~~~

### Eventos del Pallet

Después de que hayamos configurado nuestro pallet para emitir eventos, vamos a definir qué eventos:

~~~
// Los eventos de este palet.
decl_event! {
    pub enum Event<T> where AccountId = <T as system::Trait>::AccountId {
        /// Un evento emitido cuando una proof ha sido registrada
        ClaimCreated(AccountId, Vec<u8>),
        /// Un evento cuando un registro es revocada por su propietario
        ClaimRevoked(AccountId, Vec<u8>),
    }
}
~~~

Nuestro pallet solo tendrá dos eventos:
1. Cuando se añade una nueva "proof" (prueba) a la cadena de bloques.
2. Cuando se elimina una "proof" (prueba).

Los eventos pueden contener algunos metadatos, en ese caso, cada evento mostrará quién activó dicho evento (AccountId), y los datos de prueba o "proof data" (como Vec<u8>) que están siendo almacenados o eliminados.

### Pallet Storage Items (Almacenamiento de items del pallet)

Para añadir una nueva proof (prueba) a la blockchain, simplemente almacenaremos en el storage del pallet. Para almacenar este valor, crearemos un hash map relacionando la proof a el propietario de dicha proof y al número de bloque donde fue creada.

~~~
// This pallet's storage items.
decl_storage! {
    trait Store for Module<T: Trait> as PoeStorage {
        /// The storage item for our proofs.
        /// Crea un mapa que relaciona una proof con el usuario que la reclamó y cuándo la hizo.
        Proofs: map Vec<u8> => (T::AccountId, T::BlockNumber);
    }
}
~~~

Si una proof tiene un propietario y un número de bloque, entonces, sabemos que ha sido reclamada! De lo contrario, la proof está todavía disponible para ser reclamada.

## Callable Pallet Functions (Funciones llamables del Pallet)

Tal y como definimos los eventos del pallet, tendremos dos funciones que el usuario puede llamar en el pallet de Substrate:

1. create_(): Permite a un usuario reclamar la existencia de un archivo con una proof.
2. revoke_claim(): Permite al propietario revokar su reclamación.

Aquí está la declaración del pallet con las dos funciones definidas:

~~~
// Las funciones que el pallet puede realizar.
decl_module! {
    /// La declaración del módulo.
    pub struct Module<T: Trait> for enum Call where origin: T::Origin {
        // Una función por defecto para depositar eventos
        fn deposit_event() = default;

        /// Permite a un usuario reclamar la propiedad de una prueba que no haya sido reclamada
        fn create_claim(origin, proof: Vec<u8>) -> dispatch::Result {
            // Verifica que la transacción entrante está firmada y alamacena quién es el 
            // que ha llamado a la función.
            let sender = ensure_signed(origin)?;

            //Verifica que la proof especificada no ha sido reclamada todavía o si hay algún error 
            ensure!(!Proofs::<T>::exists(&proof), "This proof has already been claimed.");

            // Llama al `system` del pallet para conseguir el número de bloque
            let current_block = <system::Module<T>>::block_number();

            // Almacena la prueba con el "sender"(el que ha creado la reclamación) y el número de bloque
            Proofs::<T>::insert(&proof, (sender.clone(), current_block));

            //Emite un evento avisando de que la reclamación fue creada
            Self::deposit_event(RawEvent::ClaimCreated(sender, proof));
            Ok(())
        }

        /// Permite al propietario revocar su reclamación
        fn revoke_claim(origin, proof: Vec<u8>) -> dispatch::Result {
            // Determina quién ha llamado a la función
            let sender = ensure_signed(origin)?;

            // Verifica que que determinada proof ha sido reclamada
            ensure!(Proofs::<T>::exists(&proof), "This proof has not been stored yet.");

            // Obtiene al propietario de la reclamación
            let (owner, _) = Proofs::<T>::get(&proof);

            // Verifica que el sender (el que ha reclamado la propiedad) es el propietario de ésta
            ensure!(sender == owner, "You must own this claim to revoke it.");

            // Elimina las reclamaciones del storage
            Proofs::<T>::remove(&proof);

            //  Emite un evento cuando la reclamación se ha realizado
            Self::deposit_event(RawEvent::ClaimRevoked(sender, proof));
            Ok(())
        }
    }
}
~~~


## Compila tu nuevo Pallet

Después de haber copiado correctamente todas las partes de este pallet en template.rs, usted debería ser capaz de recompilar su nodo sin aviso ni error:

`cargo build --release`

Ahora puede reiniciar el nodo:
`#Purga la cadena para eliminar antiguos estados `
`#Te pedirá que aceptes escribiendo 'y'`
`./target/release/node-template purge-chain --dev`
`#Vuelve a ejecutar el nodo en modo "developer" o desarrollador`
`./target/release/node-template --dev`

¡Ahora es el momento de interactuar con nuestro nuevo pallet de Proof Of Existence!