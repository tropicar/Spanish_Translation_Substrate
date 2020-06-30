# Parte 4: Uso de cuentas externas

En este tutorial, no mostraremos como crear cuentas usando esta api. La razón es simple, no queremos animar a los desarrolladores a que administren las cuentas desde el navegador. Ataques de DNS, phising y otros ataques comunes ponen a los usuarios en peligro. En caso de que la DApp lo permita, sugerimos usar cuentas externas o "injected accounts". Usuarios crearan y administrarán sus cuentas de forma externa a la DApp, simplemente usaremos estas cuentas si el usario lo acepta. En esta parte, mostraremos como acceder a los balances de cuentas externas y enviar transacciones desde éstas.
4.1 Obtener la extensión Polkadot-js
Ponte en el lugar de los usuarios e instala la extensión de Pokadot-js desde las stores de Chrome o Fiferox. Crea una cuenta usando la extensión y ponle un nombre.
Ahora adaptaremos nuestra aplicación para "inject" o importar una cuenta creada de forma externa en nuestra interfaz actual.
Así es como se ve la extensión con 2 cuentas que se llaman Bob y Alice:

4.2 Mostrar balances de cuentas externas
Se puede acceder a cuentas generadas de forma externa gracias a @polkadot/extension-dapp. Vamos a añadirlo a nuestra aplicación:
yarn add @polkadot/extension-dapp@beta
Este paquete nos da acceso a:
web3Enable para declarar nuestra DApp y solicitar el derecho de leer las cuentas de nuestras usuarios.
web3Accounts/code> debería ser llamada después de llamar a web3Enable. Devolverá un array de cuentas: si los usuarios tienen una extensión compatible y si nos han permitido el acceso a sus cuentas. De lo contrario, web3Accounts devolverá un array vacío.
Para seguir haciendo las cosas de manera sencilla, no mostraremos más que una pantalla de carga o "loader" a nuestros usuarios mientras nuestros usuarios solicitan la autorización. Esto es una situación de bloqueo que no debe reproducirse en una DApp real, muestra algo útil a tus usuarios y esperan para que acepten la petición de la extensión.
Presentaremos una nueva variable de estado llamada accountLoaded para eliminar la pantalla de carga o "loader", tan pronto como, nuestros usuarios nos hayan concecido nuestros accesos. Ten en cuenta que todavía queremos permitir nuestros usuarios accedan a la aplicación incluso si no tienen la extensión, o si ellos no nos conceden el acceso. Simplemente les mostraríamos las cuentas de prueba, y nada más.
// En la parte superior de App.js
// Nuevas variables de estado para monitorizar el estado de carga de cuentas
const [accountLoaded, setaccountLoaded] = useState(false);

// en nuestra función de renderización, muestra un loader a los usuarios con una extensión compatible mientras 
// espera la autorización
if (!accountLoaded) {
return loader('Loading accounts (please review any extension\'s authorization)');
}
Declararemos nuestra DApp y solicitaremos acceso a las cuentas externas en un "hook" de React de useEffect con el que ya estamos familiarizados. Lo utilizamos para llamar loadAccounts() para inicializar el keyring. Esta función también acepta cuentas importadas o "injected" como argumentos, por lo que todo lo que tenemos que hacer es loadAccounts(injectedAccounts). Muy útil, ¿no?
En lugar de llamar a la función de loadAccounts en su función "hook" de React de useEffect, lo llamaremos una vez hayamos recolectado las cuentas externas y que los usuarios nos permitan el acceso.
// Comienzo de Apps.js
import { web3Accounts, web3Enable } from '@polkadot/extension-dapp';


// nuevo hook para obtener cuentas importadas
useEffect(() => {
web3Enable('substrate-front-end-tutorial')
.then((extensions) => {
// web3Accounts promise devuelve un array de cuentas
// o un array vacío si nuestro usuario no tiene una extensión o no nos permite 
// el acceso a ninguna de sus cuentas.
web3Accounts()
    .then((accounts) => {
        // Añade la dirección/source a el nombre para evitar confusiones
        return accounts.map(({ address, meta }) => ({
            address,
            meta: {
            ...meta,
            name: `${meta.name} (${meta.source})`
            }
        }));
    })
    // carga nuestro keyring con las nuevas cuentas importadas
    .then((injectedAccounts) => {
        loadAccounts(injectedAccounts);
    })
    .catch(console.error);
})
.catch(console.error);
}, []);

const loadAccounts = (injectedAccounts) => {
keyring.loadAll({
    isDevelopment: true
}, injectedAccounts);
setaccountLoaded(true);
};
Hemos añadido un pequeño código al de arriba, ésta parte no es obligatoria:
.then((accounts) => {
    // Añade la dirección/source al nombre para evitar confusión
    return accounts.map(({ address, meta }) => ({
        address,
        meta: {
        ...meta,
        name: `${meta.name} (${meta.source})`
        }
    }));
})
Lo que hace es extraer la información de las cuentas importadas (inyected accounts) para modificar el nombre y añadir el meta.source entre paréntesis. De esta manera, las cuentas externas de Alice y Bob serán claramente identificadas al venir desde la extensión de polkadot-js.
Debido a que todo está bien integrado en nuestro Keyring, no tenemos que tocar nuestro componente de Balances, gestionará cuentas externas como si fueran internas.
Puedes obtener la versión de este código en el directorio part-4-2:
cd part-4-2;
yarn;
yarn start;
Si ejecuta este ejemplo, se le pedirá una autorización de substrate-front-end-tutorial.

Deberías ver los saldos de tus cuentas externas debajo de las cuentas de prueba si aceptas la solicitud.

Además, si abres la consola, verás algo similar a:
web3Enable: Enabled 1 extension: polkadot-js/0.5.1 index.js:135
web3Accounts: Found 2 addresses: 5ECyNdxrwuzmsmfPJU64zMyJo8dV86hXfjjxK3PmZx1YCurj, 5ExttMT4rtnYJ7TLn19d8C8sVdTH8exj7pxWPC5xepz7J9KF
4.3 Enviar fondos desde una cuenta externa
Aunque el componente de Balances no tiene que modificarse para atender cuentas externas, nuestro componente Transfer necesitará algunas modificaciones. La razón es simple, nuestra DApp no tiene el par de claves criptográficas de las cuentas externas. Éstos son gestionados de forma más segura por una extensión y no tendrán acceso a la clave privada. Para realizar una transferencia, tendremos que enviar la transacción a la extensión; entonces se le pedirá al usuario que la firme.
Para detectar si una cuenta es externa, podemos leer el parámetro meta.isInjected de una cuenta en nuestro Keyring. Este valor será true para las cuentas que sean externas. La api nos permite establecer o elegir un signer (firmante) para una transacción. El signer o firmante para la extensión puede ser recogido del promise de web3FromSource.
El código que necesita ser modificado está en la función makeTransfer, de nuestro Transfer.js
import { web3FromSource } from '@polkadot/extension-dapp';

const { addressTo, addressFrom, amount } = formState;
const fromPair = keyring.getPair(addressFrom);
// Extraer isInjected de los metadatos del keyring
const { address, meta: { source, isInjected } } = fromPair;
let fromParam;

// Configurar el signer o firmante
if (isInjected) {
    const injected = await web3FromSource(source);
    fromParam = address;
    api.setSigner(injected.signer);
} else {
    fromParam = fromPair;
}

setStatus('Sending...');

api.tx.balances
.transfer(addressTo, amount)
.signAndSend(fromParam, ({ status }) => {
    ...
Eso es todo, ahora puedes enviar fondos desde una cuenta externa!
Puedes obtener la versión de este código en el directorio part-4-3:
cd part-4-3;
yarn;
yarn start;
Si ejecuta este ejemplo y envía fondos desde una cuenta externa, se le solicitará una autorización desde substrate-front-end-tutorial.

4.4 Es bueno saber
Si juegas con este DApp y envías, por ejemplo, 1 "unit" de Alice a tu cuenta recién creada, te darás cuenta de varias cosas. En primer lugar, la cuenta de Alice disminuye su valor en más de una unidad. Probablemente lo has conseguido, has pagado comisiones por la transferencia.
Lo más curioso viene ahora, la transacción ha sido un éxito, pero el balance de Bob permanece sin cambios. No es un error, es una característica, la explicación es la siguiente:
El nodo de Substrate al que estamos consultando tiene un pallet "Balances". Este pallet es el responsable de hacer la traza de todas las cuentas de la blockchain. En nuestra blockchain solo hay un nodo desarrollador y éste se resetea cada hora, pero imagina una blockchain en producción a la que cualquiera pudiese acceder. Ahora imagina que esta blockchain se vuelve popular y se crean muchas cuentas, algunas de ellas contendrían una fracción de un centavo de valor. La blockchain terminaría usando muchísimo almacenamiento (storage) en cuentas inútiles, lo que provocaría un atasco y ralentizaría las operaciones de lectura/escritura. Para combatir ésto, el pallet de Balances de nuestro nodoo contiene algo llamado Existential Deposit a.k.a ED. Si una cuenta tiene menos fondos que el ED, está cuenta será eliminada completamente del state. En un nodo de desarrollo --dev, el ED es de 100'000'000. Así que asegúrese de enviar un gran número de units si quiere que las cosas cambien.
Además, si transfiere fondos a una cuenta recién creada, se aplicará una tasa llamada tasa de creación (creation fee). Para tener una visión general de todos los fees o tasas que se pueden aplicar, hay una consulta "derive query" para ello llamada: api.derive.balances.fees!
4.5 Más adelante - Extraer el botón de enviar en su propio componente
La transacción que realizamos anteriormente usando api.tx.balances.transfer. La función de transferencia se encuentra en el pallet de Balances. En el futuro, es muy probable que utilices métodos diferentes de los pallets de FRAME. Por lo tanto, una buena idea sería extraer la lógica de la función de makeTransfer en su propio componente y hacerlo genérico para que cualquier método pueda usarlo.
Este ejercicio se deja en manos del lector.
Puedes obtener la versión de este código en el directorio part-4-5:
cd part-4-5;
yarn;
yarn start;
