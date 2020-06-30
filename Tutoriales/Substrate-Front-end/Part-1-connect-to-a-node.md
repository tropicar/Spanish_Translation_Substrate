
# Parte 1: Conectarse a un nodo

## Framework y herramientas

En este tutorial, crearemos una aplicación React. Para centrarnos tanto como sea posible en el uso real de la APi de Polkadot-js (mirar en su documentación), no queremos dedicar demasiado tiempo en hacer que la aplicación sea bonita. Por lo que, usaremos el paquete semantic-ui-react (ver su documentación) para renderizar los componentes de la interfaz de usuario para nosotros.
Construir una aplicación encima de Substrate significa que necesitamos conectarnos a un nodo Substrate. Para empezar lo más rápido posible con el desarrollo de nuestra app, esta guía le proporcionará un endpoint de desarrollo la que conectar su aplicación. Para ser más flexible y ejecutar su propio nodo, por favor lea Installing Substrate.

El nodo proporcionado en este tutorial se puede obtener en wss://dev-node.substrate.dev:9944, se ejecuta con la opción --dev, y su base de datos se reinicia cada hora, así que no se sorprenda si los datos de las cuentas se resetean. La ventaja de tener tu propio nodo es que puedes leer los errores del "runtime" en la consola y tener control total en la configuración de la blockchain.

Esta aplicación ha sido creada usando el comando create-react-app, el cúal crea el esqueleto de una aplicación. Usaremos el gestor de paquetes de yarn a lo largo de esta guía. Clona el siguiente repositorio substrate-front-end :

git clone https://github.com/substrate-developer-hub/substrate-front-end.git
cd substrate-front-end

Después de eliminar el "boilerplate code" (código que se repite en la aplicación), añadiremos nuestras primeras dependencias al repositorio:@polkadot/api@beta, semantic-ui-react, semantic-ui-css

El código de Substrate evoluciona muy rápidamente y regularmente hay cambios importantes. La API Polkadot-js sigue el mismo camino, por lo que algunas cosas dejan de funcionar. Usaremos las versiones beta de todos los módulos de Polkadot-js para que la DApp que vamos a construir sea básica y se adapte a los futuros cambios más fácilmente, requiriendo así menos trabajo. Sin embargo, se aconseja utilizar las versiones estables para cualquier aplicación en producción.
Para empezar, conectaremos nuestra aplicación a nuestro nodo de desarrollo usando los métodos incorporados de polkadot/api. A continuación, pediremos información a nuestro nodo usando los métodos de rpc.system que están disponibles en la api.

1.2 Conectar un nodo

Vaya al archivo src/App.js para empezar. Empezamos por importar los siguientes paquetes:

~~~
import { ApiPromise, WsProvider } from '@polkadot/api';
import React, { useState, useEffect } from 'react';
import { Container, Dimmer, Loader} from 'semantic-ui-react';
~~~

import 'semantic-ui-css/semantic.min.css'
Esta aplicación hará uso de "React hooks". Ello nos permitirá usar propiedades como el estado y otras más características de React sin tener que escribir una clase. WsProvider y ApiPromisede la API de Polkadot-js nos permiten conectarnos a un nodo y exponer las funciones del nodo en nuestra aplicación. Comencemos por conectar el nodo en el puerto WebSocket por defecto 9944.
  const [api, setApi] = useState();
  const [apiReady, setApiReady] = useState();
  const WS_PROVIDER = 'wss://dev-node.substrate.dev:9944';

  useEffect(() => {
    const provider = new WsProvider(WS_PROVIDER);

    ApiPromise.create(provider)
      .then((api) => {
        setApi(api);
        api.isReady.then(() => setApiReady(true));
      })
      .catch((e) => console.error(e));
  }, []);
El primer "hook" que vamos a usar es useeffect. Esta función se activará cuandonuestros componentes de la App carguen, tal y como componentDidMount haría.
Lo primero que necesitas hacer es definir tu proveedor, en nuestro caso, el nodo de desarrollo wss://dev-node.substrate.dev:9944. Si desea conectarse a un nodo local, puede establecerlo mediantews://127.0.0.1:9944.
Luego crearemos una variable llamada api que devolverá ApiPromise.create. Añadiremos dicho valor al estado para que sea más fácil acceder a él, porque tendrémos que usarlo muchas veces para interactuar con nuestro nodo.
Para saber si nuestra aplicación está conectada de manera correcta al nodo, lo comprobaremos usando la función promise que nos permite api llamado isReady, el cuál, avisará cuando la conexión haya sido establecida. Tan pronto como devuelva el valor, cambiaremos el valor de apiReady. Ya que nuestra aplicación no podrá acceder a ninguna información del nodo o de la blockchain antes de que la conexión se haya establecido, mostraremos un sistema de carga a los usuarios para informarles de que estamos conectándonos a un nodo.
Si eres nuevo en el uso de los hooks de React, será interesante para ti que el primer argumento de useEffect es la función que será capturada (en este caso, crear la conexión al nodo). El segundo argumento que estamos pasando es un array para decirle a nuestro hook cuándo debe ser activado de nuevo. Verá claramente lo qué es. En este caso, nuestro api.create solo debe ejecutarse una vez. Como no será necesario actualizar el estado en el futuro, pasamos un array vacío.
Así es como renderizaremos el contenido.
  const loader = function (text){
    return (
      <Dimmer active>
        <Loader size='small'>{text}</Loader>
      </Dimmer>
    );
  };

  if(!apiReady){
    return loader('Connecting to the blockchain')
  }

  return (
    <Container>
      Connected
    </Container>
  );
Como se describió anteriormente, este código verificará que la el estado de la api está inicializado y preparado antes de mostrar a nuestro usuarios que está "conectado". El loader o función de carga mostrará un bonito loader (pantalla de carga) con el texto que queramos. La lógica del "loader" está fuera de su propia función porque la reutilizaremos más adelante en el tutorial.
Puedes obtener el código visitando el directorio part-1-2 y ejecutándolo con:
cd part-1-2;
yarn;
yarn start;
Deberías ver el cargador durante un par de segundos y luego "Conectado" 🚀.
1.3 Obteniendo información del nodo
Ahora que estamos conectados a nuestro nodo, vamos a mostrar información sobre éste. Crearemos un nuevo componente llamado NodeInfo, el cuál será el responsable de mostrar esta información. Pasaremos el objeto de la api que habíamos añadido al estado del componente de la App a los accesorios de este componente. De esta manera, tendremos acceso a todos los métodos y funciones del nodo Substrate al que estamos conectados.
Todo lo que necesitamos es importar el componente en React y las funciones "Hook".
import React, {useEffect, useState} from 'react';
La api de Polkadot-js permite enviar llamadas RPC al nodo. Para obtener la información del nodo que queremos, usaremos los métodos rpc.system que nos ofrece la api. Añadiremos en el estado del componente la información que nos devuelva la api para poder acceder a ella y mostrarla en el futuro.
export default function NodeInfo(props) {
  const {api} = props;
  const [nodeInfo, setNodeInfo] = useState({})

  useEffect(() => {
    Promise.all([
      api.rpc.system.chain(),
      api.rpc.system.name(),
      api.rpc.system.version(),
    ])
    .then(([chain, nodeName, nodeVersion]) => {
      setNodeInfo ({
        chain,
        nodeName,
        nodeVersion
      })
    })
    .catch((e) => console.error(e));
  },[api.rpc.system]);
¿Recuerda lo que le dijimos en la parte anterior? El segundo parámetro de useEffect es un array que muestra las variables que queremos representar. En este caso, se ejecutará la función useEffect cuando el componente se cargue y cuando api.rpc.system cambie.
El "endpoint" de la api (punto de acceso de la api) no cambia a no ser que haya una actualización del "runtime" del nodo. Aunque ésto, no ocurrirá en nuestro entorno de desarrollo y el componente funcionará sin especificar este argumento, todavía estamos enviando el api.rpc.system a useEffect para hacer que nuestros ejemplos sean lo más realista posible.
Para finalizar, mostraremos la información de nuestro nodo en la parte superior de nuestra página:
  return (
    <>
      {nodeInfo.chain} - {nodeInfo.nodeName} (v{nodeInfo.nodeVersion})
      <hr/>
    </>
  )
}
Puedes obtener el código en el directorio part-1-3 y ejecutándolo con:
cd part-1-3;
yarn;
yarn start;
Debería aparecer algo como: "Development - substrate-node (v2.0.0)"