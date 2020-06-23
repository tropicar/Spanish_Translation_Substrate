# Parte 2: Consulta de balances

Ahora que tenemos conexión con nuestro nodo, vamos a jugar con algunas cuentas. Como estamos conectados a un nodo que ha sido ejecutado con la opción --dev, podemos acceder a las cuentas predefinidas cuyas claves privadas conocemos. Las cuentas están organizadas en algo llamado "keyring". Desde este "keyring", puedes acceder a las direcciones de una cuenta, a su nombre asociado y a su par de claves criptográfica (clave pública y privada) si esta cuenta es gestionada localmente. El paquete @polkadot/ui-keyring contiene bastantes utilidades para gestionar cuentas. Haremos uso de ellas en las siguientes secciones.
En esta parte usaremos el método api.query para acceder al "storage" o almacenamiento del componente de balances de la blockchain.
Obtener cuentas de prueba (testing accounts)
Primero de todo, importaremos el objeto keyring; Necesitaremos añadir las dependencias @polkadot/keyring, @polkadot/ui-identicon, @polkadot/ui-keyring a nuestro proyecto para poder utilizar las utilidades que nos ofrece keyring.
import keyring from '@polkadot/ui-keyring';
Inicializar "keyring" es sencillo, solo tenemos que usar loadAll(). Para acceder a las cuentas de testing, pasaremos el objeto {isDevelopment: true} como parámetro. Añadiremos una nueva función useEffect para inicializar nuestro "keyring". Usamos console.log para inspeccionar que nos devuelve.
Éste es el código que hemos añadido:
  useEffect(() => { 
    keyring.loadAll({
      isDevelopment: true
    });

    console.log(keyring);
  },[]); 
Y ésto es lo que debería mostrar por consola:
{…}
​_accounts: Object { add: add(), remove: remove(), subject: {…} }
​_addresses: Object { add: add(), remove: remove(), subject: {…} }
​_contracts: Object { add: add(), remove: remove(), subject: {…} }
​_genesisHash: undefined
​_keyring: Object { _pairs: {…}, _type: "ed25519", decodeAddress: decode(), … }
​_prefix: undefined
​_store: Object {  }
​decodeAddress: function decodeAddress()​
encodeAddress: function encodeAddress()​
stores: Object { address: address(), contract: contract(), account: account() }
Si expandes los objetos de keyring y los objetos pairs, verás que tenemos algunos pares de claves aquí. Éstas son las cuentas de testing que habíamos inicializado gracias a la opción is Development. Ahora crearemos un componente de balances que nos permita mostrar esas cuentas. Este componente de balances mostrará rápidamente todas nuestras cuentas así como la cantidad de fondos de cada una de ellas.
Emperezaremos listando los nombres de las cuentas y sus direcciones. Como hablamos anteriormente,getPairs() nos devolverá un array de las cuentas en nuestro keyring. El mapear este array nos permitirá mostrar el nombre y la dirección de cada una de las cuentas.
import React from 'react';
import { Table } from 'semantic-ui-react';

export default function Balances (props) {
  const { keyring } = props;
  const accounts = keyring.getPairs();

  return (
    <>
      <h1>Balances</h1>
      <Table celled striped>
        <Table.Body>
          {accounts.map((account, index) => {
            return (
              <Table.Row key={index}>
                <Table.Cell textAlign='right'>{account.meta.name}</Table.Cell>
                <Table.Cell>{account.address}</Table.Cell>
              </Table.Row>
            );
          })}
        </Table.Body>
      </Table>
    </>
  );
}
Por supuesto que necesitamos modificar el punto de entrada (entry point) de nuestra DApp, App.js, para importar este nuevo componente y renderizarlo. Ten en cuenta que pasamos la api como parámetro, ésto no es necesario en este momento, pero lo usaremos más adelante.
// En la parte superior del archivo
import Balances from './Balances';

// in the return, after NodeInfo
<Balances
  api={api}
  keyring={keyring}
/>
Puedes obtener la versión de este código en el directorio part-2-1:
cd part-2-1;
yarn;
yarn start;
Si ejecuta este ejemplo, obtendrás una tabla con "Alice", "Alice_stash", "Bob", y sus direcciones.

2.2 Consultar y mostrar el balance de Alice
Ahora que tenemos nuestras cuentas, podemos hacer una petición o "query" del balance de cada una de ellas. El nodo Substrate al que estás conectado dispone de un pallet llamado Balances que tiene la traza (tracking) del balance de cada una de las cuentas de la blockchain. Miraremos el freeBalance de una cuenta. La API de Polkadot-js permite realizar peticiones al storage de un pallet con api.query y realizar las actualizaciones de estos valores. Así es como podemos obtener acceso a la información y obtener el freeBalance.
Vamos a ir poco a poco por realizar una petición o query del balance de una cuenta, la de Alice.
export default function Balances (props) {
  const { api, keyring } = props;
  const accounts = keyring.getPairs();
  // Este estado obtendrá la trazabilidad del freeBalance de Alice , es decir, //seguirá todos los cambios del freeBalance
  const [balance, setBalance] = useState('0');
  useEffect(() => {
    let unsubscribe;

    api.query.balances.freeBalance(accounts[0].address, (currentBalance) => {
        setBalance(currentBalance.toString());
      })
      .then(unsub => { unsubscribe = unsub; })
      .catch(console.error);

    return () => unsubscribe && unsubscribe();
  }, [accounts, api.query.balances, api.query.balances.freeBalance]);
Empezamos creando una variable de estado llamada balance, que iniciaremos a 0. Entonces, como dijimos en la parte anterior, realizaremos la petición usando un "Hook" de react con la función useEffect. Queremos mostrar el balance y actualizar todos los cambios aceptados por el nodo de Substrate.

Subscribe (Subscribirse)
El método de la API api.query.balances.freeBalance() toma como primer argumento la dirección de una cuenta, y puedes pasar como segundo argumento opcional una función de callback para actualizar cualquier novedad. La función callback escucha los cambios del valor de freeBalance de la cuenta que introduzcas.
Estamos actualizando el estado del componente con los valores que estamos obteniendo de la API y que nos permite ver los cambios en el balance sin recargar la página!

Unsubscribe (desubscribirse)
Pero debido a que es una subscripción, debemos darnos de bajo cuando de desmonte el componente de React de Balances. La función unsubscribe es lo que devuelve la query o petición. Ésta debe ser llamada en el return de la función useEffect que será creado cuando el componente no esté "unmount" o desmontado. En nuestro caso, estará "unmount" cuando cerrremos la pestaña o refresquemos nuestra aplicación de forma manual.
Vamos a desmeruzar la tabla que nos devuelve para mostrar solamente el balance de Alice.
  // Final del archivo Balance.js

  return (
    <>
      <h1>Balances</h1>
      <Table celled striped>
        <Table.Body>
          <Table.Row key={0}>
            <Table.Cell textAlign='right'>{accounts[0].meta.name}</Table.Cell>
            <Table.Cell>{accounts[0].address}</Table.Cell>
            <Table.Cell>{balance}</Table.Cell>
          </Table.Row>
        </Table.Body>
      </Table>
    </>
  );
}
Puedes obtener la versión de este código en el directorio part-2-2:
cd part-2-2;
yarn;
yarn start;
Si ejecutas este ejemplo, obtendrás una tabla con la cuenta y el balance de Alice.

2.3 Consultar y mostrar todos los balances
Ahora que sabemos cómo funciona el consultar el estado de un pallet, podemos mapear todas las cuentas y consultar el estado de cada una de ellas. La api tiene una función útil para emparejar múltiples consultas al mismo tiempo; se le conoce como multi. Puede pasar un array de argumentos y devolverá los resultados como un array.
En lugar de tener solamente el balance de Alice en el estado, lo convertiremos en un mapa entre address y freeBalance, para así poder usarlo para mostrar el balance de cada cuenta. Dado que el multi sigue siendo una subscripción, la lógica para cancelar la subscripción es la misma.
const { api, keyring } = props;
const accounts = keyring.getPairs();
const addresses = accounts.map(account => account.address);
const accountNames = accounts.map((account) => account.meta.name);
const [balances, setBalances] = useState({});

useEffect(() => {
  let unsubscribeAll

  api.query.balances.freeBalance
    .multi(addresses, (currentBalances) => {
      const balancesMap = addresses.reduce((acc, address, index) => ({
        ...acc,
        [address]: currentBalances[index].toString()
      }), {});
      setBalances(balancesMap);
    })
    .then(unsub => { unsubscribeAll = unsub; })
    .catch(console.error);

  return () => unsubscribeAll && unsubscribeAll();
}, [addresses, api.query.balances.freeBalance, setBalances]);
Llamamos a api.query.balances.freeBalance.multi y pasamos nuestro array de direcciones, entonces, el callback recibe un array de los resultados. A partir de aquí, creamos un objeto con reduce que contiene la dirección y el balance de cada cuenta. Este objeto es el que persiste en el estado.
Mostrar las cuentas con sus balances es ahora muy fácil. Mapearemos nuestra lista de direcciones y mostrar los nombres y balances.
  return (
    <>
      <h1>Balances</h1>
      <Table celled striped>
        <Table.Body>
          {addresses.map((address, index) => {
            return (
              <Table.Row key={index}>
                <Table.Cell textAlign='right'>{accountNames[index]}</Table.Cell>
                <Table.Cell>{address}</Table.Cell>
                <Table.Cell>{balances && balances[address]}</Table.Cell>
              </Table.Row>
            );
          })}
        </Table.Body>
      </Table>
    </>
  );
Return ( <>
Balances
{addresses. map((address, index) => { return ( {accountNames[index]} {address} {balances && balances[address]} ); })}
):
cd part-2-3;
yarn;
yarn start;
Si ejecuta este ejemplo, obtendrá una tabla con todas nuestras cuentas de pruebas, sus direcciones y sus saldos.

2.4 Es bueno saber
Vale la pena mencionar que los fondos en freeBalance de una cuenta pueden no estar disponibles para ser transferidos. Una cuenta podría tener fondos, pero si, por ejemplo, éstos están "staked" (a.k.a bonded) para validar, puede que no no estén disponibles para transferirlos incluso si el freeBalance no es null. Si quieres mostrar el balance disponible para transacciones, ésto es computado en api-derive. Echa un vistazo al campo disponible devuelto por derive.balances.all.
La derive api, como su nombre sugiere, no es una consulta directa al nodo. Es más bien una concatenación y una derivación de múltiples consultar para servir información que no es directamente accesible desde el nodo. Una función popular es derive.chain.bestNumber para obtener el último número de bloque. Echa un vistazo en el repositorio. Lamentablemente, estos endpoints (puntos de acceso) de la api están solo documentados en el código, por ahora.
Substrate es un framework para crear blockchains. El nodo --dev que estamos consultando en esta parte contiene un pallet llamado Balances de FRAME, el cuál tiene la traza del balance de cada una de las cuentas de la blockchain. No todas las chain de Substrate tendrán un pallet de Balances. Sin embargo, cualquier Pallet que esté integrado en nuestro nodo Substrate puede ser consultado usando api.query.section.method.
