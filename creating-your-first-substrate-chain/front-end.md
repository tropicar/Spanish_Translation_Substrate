# Front-End

Este tutorial no es sobre crear interfaces de usuario, pero queremos mostrar cómo de fácil es crearlas con Substrate. Si has llegado hasta aquí, quiere decir que ya tienes blockchain personalizada en funcionamiento.

Le daremos un componente personalizado creado en React, el cúalm, podrás añadir a tu código substrate-front-end-template para poder interactuar con tu nodo.

## Añade tu componente de React personalizado

En el directorio substrate-front-end-template del proyecto, modifique el archivo TemplateModule.js en la carpeta /src/:

~~~
substrate-front-end-template
|
+-- src
|   |
|   +-- index.js
|   |
|   +-- App.js
|   |
|   +-- TemplateModule.js  <-- Modifica este archivo
|   |
|   +-- ...
+-- ...
~~~

Elimina el contenido de ese archivo y en su lugar usa el siguiente componente.

~~~
```js // React and Semantic UI elements. import React, { useState, useEffect } from 'react'; import { Form, Input, Grid, Message } from 'semantic-ui-react'; // Herramientas precompiladas de Substrate front-end para conectarse a un nodo // y hacer una transacción. import { useSubstrate } from './substrate-lib'; import { TxButton } from './substrate-lib/components'; //Utilidades de Polkadot-JS para hashing data. import { blake2AsHex } from '@polkadot/util-crypto'; // Nuestro componente principal de Proof Of Existence que será exportado. export default function ProofOfExistence (props) { //Establecer una API para comunicarte con nuestro nodo Substrate. const { api } = useSubstrate(); // Consigue el "usuario seleccionado" de el componente `AccountSelector`. const { accountPair } = props; // React hooks for all the state variables we track. // Learn more at: https://reactjs.org/docs/hooks-intro.html const [status, setStatus] = useState(''); const [digest, setDigest] = useState(''); const [owner, setOwner] = useState(''); const [block, setBlock] = useState(0); // Our `FileReader()` which is accessible from our functions below. let fileReader; // Takes our file, and creates a digest using the Blake2 256 hash function. const bufferToDigest = () => { // Turns the file content to a hexadecimal representation. const content = Array.from(new Uint8Array(fileReader.result)) .map(b => b.toString(16).padStart(2, '0')) .join(''); const hash = blake2AsHex(content, 256); setDigest(hash); }; // Callback function for when a new file is selected. const handleFileChosen = file => { fileReader = new FileReader(); fileReader.onloadend = bufferToDigest; fileReader.readAsArrayBuffer(file); }; // React hook to update the "Owner" and "Block Number" information for a file. useEffect(() => { let unsubscribe; // Polkadot-JS API query to the `proofs` storage item in our pallet. // This is a subscription, so it will always get the latest value, // even if it changes. api.query.templateModule .proofs(digest, result => { // Our storage item returns a tuple, which is represented as an array. setOwner(result[0].toString()); setBlock(result[1].toNumber()); }) .then(unsub => { unsubscribe = unsub; }); return () => unsubscribe && unsubscribe(); // This tells the React hook to update whenever the file digest changes // (when a new file is chosen), or when the storage subscription says the // value of the storage item has updated. }, [digest, api.query.templateModule]); // We can say a file digest is claimed if the stored block number is not 0. function isClaimed () { return block !== 0; } // The actual UI elements which are returned from our component. return (

Proof Of Existence

{/* Show warning or success message if the file is or is not claimed. */}
{/* File selector with a callback to `handleFileChosen`. */}  {/* Show this message if the file is available to be claimed */} {/* Show this message if the file is already claimed. */} {/* Buttons for interacting with the component. */} {/* Button to create a claim. Only active if a file is selected, and not already claimed. Updates the `status`. */} {/* Button to revoke a claim. Only active if a file is selected, and is already claimed. Updates the `status`. */}{/* Status message about the transaction. */}
{status}
); } ```
~~~ 

No vamos a ir paso a paso a través de toda la creación del componente, pero revisa los comentarios del código para aprender lo que cada parte hace.

## Subir una Proof

Su front-end debería reiniciarse automáticamente con este nuevo componente incluido. Seleccione cualquier archivo de su ordenador, y verá que puede crear una reclamación con el "digest" o resumen del archivo:

Si presiona en "Create Claim", se enviará una transacción al Pallet de Proof Of Existence que has creado, dónde el "digest" (resumen) y la cuenta del usuario seleccionada se almacenarán en la cadena.

Si todo ha ido bien, deberías ver un nuevo evento ClaimCreated en el componente de eventos. El front-end puede automáticamente reconocer que tu archivo ha sido registrado, e incluso te permite la opción de revocar el registro si tu quieres.

¡Recuerde, sólo el propietario puede revocar la reclamación! Si selecciona otra cuenta de usuario en la parte superior, verá que la opción de revocación está desactivada!

## Siguientes Pasos

Este es el final de nuestro viaje en la creación de una blockchain de Proof Of Existence.

Has visto de primera mano lo simple que puede ser desarrollar un nuevo Pallet y ponerlo en ejecución usando Substrate. Además, te hemos mostrado el ecosistema Substrate, así como las herramientas que te proporciona para crear de forma rápida un front-end responsive para que los usuarios puedan interactuar con tu blockchain.

En este tutorial se han omitido algunos detalles específicos en el desarrollo con el objetivo de que era corto y satisfactorio. Sin embargo, queremos que sigas aprendiendo!

Si estás interesado en aprender a programar tu propio palet en Substrate, por favor prueba nuestro Substrate Collectables Workshop. Este tutorial te enseñará paso a paso cómo programar tus propios palets personalizados. Además, te iniciará en los conceptos de desarrollo avanzados del runtime y en las buenas prácticas a la hora de crear con Substrate.

Es hora de decir que tu éxito usando el framework Substrate estará limitado a la habilidad que tengas programando en Rust. El [Rust Book][Rust Book] es el mejor punto de partida para desarrolladores de cualquier experiencia.

Si encuentras algún problema con este tutorial o quiere mostrarnos sugerencías u apoyarnos, tómate la libertad de abrir una issue de Github con tus ideas.

¡No podemos esperar a ver lo que creas a continuación!

[Rust Book]:https://doc.rust-lang.org/book/