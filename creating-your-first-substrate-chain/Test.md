# Testeando tu nueva Blockchain

Este tutorial no es sobre crear interfaces de usuario, pero queremos mostrar cómo de fácil es crearlas con Substrate. Si has llegado hasta aquí, quiere decir que ya tienes blockchain personalizada en funcionamiento.

Le daremos un componente personalizado creado en React, el cúalm, podrás añadir a tu código substrate-front-end-template para poder interactuar con tu nodo.

¡Vamos a probarlo!

## Añade tu componente de React personalizado

En el proyecto substrate-front-end-template, crea un archivo llamado ProofOfExistence.jsx in la carpeta /src/:

~~~
substrate-front-end-template
|
+-- src
|   |
|   +-- index.jsx
|   |
|   +-- App.jsx 
|   |
|   +-- TemplateModule.jsx  <-- Edita este archivo
|   |
|   +-- ...
+-- ...
~~~

En ese archivo, reemplace el contenido existente por el siguiente componente.

~~~
import React, { useState, useEffect } from "react";
import { Form, Input, Grid, Message } from "semantic-ui-react";
import { blake2AsHex } from "@polkadot/util-crypto";

import { useSubstrate } from "../substrate-lib";
import { TxButton } from "../substrate-lib/components";

export default function ProofOfExistence(props) {
  const { api } = useSubstrate();
  const { accountPair } = props;
  const [status, setStatus] = useState("");
  const [digest, setDigest] = useState("");
  const [owner, setOwner] = useState("");
  const [block, setBlock] = useState(0);

  let fileReader;

  const bufferToDigest = () => {
    const content = Array.from(new Uint8Array(fileReader.result))
      .map(b => b.toString(16).padStart(2, "0"))
      .join("");

    const hash = blake2AsHex(content, 256);
    setDigest(hash);
  };

  const handleFileChosen = file => {
    fileReader = new FileReader();
    fileReader.
    fileReader.readAsArrayBuffer(file);
  };

  useEffect(() => {
    let unsubscribe;

    api.query.templateModule
      .proofs(digest, result => {
        setOwner(result[0].toString());
        setBlock(result[1].toNumber());
      })
      .then(unsub => {
        unsubscribe = unsub;
      });

    return () => unsubscribe && unsubscribe();
  }, [digest, api.query.templateModule]);

  function isClaimed() {
    return block !== 0;
  }

  return (
    <Grid.Column>
      <h1>Proof Of Existence</h1>
      <Form success={!!digest && !isClaimed()} warning={isClaimed()}>
        <Form.Field>
          <Input
            type="file"
            id="file"
            label="Your File"
             => handleFileChosen(e.target.files[0])}
          />
          <Message success header="File Digest Unclaimed" content={digest} />
          <Message
            warning
            header="File Digest Claimed"
            list={[digest, `Owner: ${owner}`, `Block: ${block}`]}
          />
        </Form.Field>

        <Form.Field>
          <TxButton
            accountPair={accountPair}
            label={"Create Claim"}
            setStatus={setStatus}
            type="TRANSACTION"
            attrs={{ params: [digest], tx: api.tx.templateModule.createClaim }}
            disabled={isClaimed() || !digest}
          />
          <TxButton
            accountPair={accountPair}
            label="Revoke Claim"
            setStatus={setStatus}
            type="TRANSACTION"
            attrs={{ params: [digest], tx: api.tx.templateModule.revokeClaim }}
            disabled={!isClaimed() || owner !== accountPair.address}
          />
        </Form.Field>
        <div style={{ overflowWrap: "break-word" }}>{status}</div>
      </Form>
    </Grid.Column>
  );
}
~~~

No vamos a ir paso a paso en la creación de este componente, pero echa un vistazo a los comentarios del código para aprender qué hace cada parte.

## Subir una Proof

Tu front-end debería haberse reiniciado automáticamente una vez hayas incluido este nuevo componente.Selecciona cualquier archivo de tu ordenador, y verás que puedes crear una reclamación (claim) con la digest o resumen del archivo:

Si pulsas en "Create Claim", se enviará una transacción en tu módulo personalizado de Proof Of Existence, dónde el digest y la cuenta de usuario seleccionada serán almacenadas en la blockchain.

Si todo va bien, deberías ver un nuevo evento ClaimCreated aparecer en el componente Events. El front-end puede automáticamente reconocer que tus archivos están ahora registrados, y ello te permite la opción de revocar dicho registro si tú quieres.

¡Recuerde, sólo el propietario puede revocar la reclamación! Si selecciona otra cuenta de usuario en la parte superior, verá que la opción de revocación está desactivada!

## Siguientes Pasos

Este es el final de nuestro viaje en la creación de una blockchain de Proof Of Existence.

Has visto de primera mano lo simple que puede ser desarrollar un nuevo Pallet y ponerlo en ejecución usando Substrate. Además, te hemos mostrado el ecosistema Substrate, así como las herramientas que te proporciona para crear de forma rápida un front-end responsive para que los usuarios puedan interactuar con tu blockchain.

En este tutorial se han omitido algunos detalles específicos en el desarrollo con el objetivo de que sea corto y entretenido de hacer. Sin embargo, ¡queremos que sigas aprendiendo!

Si estás interesado en aprender a programar tu propio pallet en Substrate, por favor prueba nuestro Substrate Collectables Workshop. Este tutorial te enseñará paso a paso cómo programar tus propios módulos en el runtime. Además, te iniciará en los conceptos de desarrollo avanzados del runtime y en las buenas prácticas a la hora de crear con Substrate.

Es hora de decir que tu éxito usando el framework Substrate estará limitadopor tu habilidad programando en Rust. El Rust Book es el mejor punto de partida para desarrolladores de cualquier experiencia.

Si encuentras algún problema con este tutorial, quiere mostrarnos sugerencias o apoyarnos, tómate la libertad de abrir una issue de Github con tus ideas.

¡No podemos esperar a ver lo que crear a continuación!