# Parte 3: Transferencia de fondos
Ahora que hemos mostrados todos los balances de nuestras cuentas, hagamos que se muevan los fondos. En esta parte, vamos a crear un componente React llamado Transfer que nos permitirá enviar fondos desde una cuenta de la que tengamos la clave privada hacia otra cuenta. Tendremos un campo denominado "from"(desde u origen), y otro "to"(destino o 'a'), y un botón para transferir.
En esta parte usaremos el método api.tx para subir una transacción en el pallet de balances de blockchain.
3.1 Transferir fondos
Debido a que solo puedes enviar fondos desde tus propias cuentas, el campo "from" será un desplegable (dropdown). El receptor de nuestros fondos puede ser cualquier dirección válida.
En este componente, no necesitamos obtener ningún dato de la api, por lo que no usaremos ningún "hook", sin embargo, haremos uso de la api para realizar la transferencia. Al igual que para Balances, pasaremos las variables api y keyring a props(accesorios). Solo tendremos solo una variable de estado, un objeto que contiene la información del formulario para ser enviada cuando hagamos click en el botón Send (enviar).
Para generar las opciones desde el menú desplegable de keyringOptions de nuestra cuenta, simplemente recorreremos o realizaremos iteraciones desde la función keyring.getPairs().
export default function Transfer (props) {
  const { api, keyring } = props;
  const [status, setStatus] = useState('');
  const initialState = {
    addressFrom: '',
    addressTo: '',
    amount: 0
  };
  const [formState, setFormState] = useState(initialState);

  //Obtiene la lista de cuentas de las que poseemos las claves privadas
  const keyringOptions = keyring.getPairs().map((account) => ({
    key: account.address,
    value: account.address,
    text: account.meta.name.toUpperCase()
  }));
Ahora vamos a crear nuestra función para subir el formulario. Para ello, necesitaremos crear una transacción, entonces, la firmaremos con nuestra clave privada para, finalmente, enviarla. La api proporciona todo lo que necesitamos una vez más, con los métodos api.tx.balances, transfer y signAndSend. Éste último toma como primer parámetro el par de claves y como segundo una función callback.
El status() (estado) de la transacción es pasado usando la función callback. La usaremos para ver si la transacción se ha ejecutado con éxito. En el siguiente ejemplo, mostraremos a nuestros usuarios si la transacción se ha realizado satisfactoriamente, comprobando el valor del estado con isFinalized. Si fue ejecutado con éxito, mostraremos en qué bloque finalizó la transacción.
const makeTransfer = () => {
    const { addressTo, addressFrom, amount } = formState;
    const fromPair = keyring.getPair(addressFrom);

    setStatus('Sending...');

    api.tx.balances
    .transfer(addressTo, amount)
    .signAndSend(fromPair, ({ status }) => {
        if (status.isFinalized) {
        setStatus(`Completed at block hash #${status.asFinalized.toString()}`);
        } else {
        setStatus(`Current transfer status: ${status.type}`);
        }
    }).catch((e) => {
        setStatus(':( transaction failed');
        console.error('ERROR:', e);
    });
Eso es todo, ¿no es una completa locura no? De la misma manera que el componente Balances, neecesitamos importarlo en Apps.js, y decir, dónde debe ser renderizado.
//En la parte superior del archivo
import Transfer from './Transfer';

//Justo después de nuestro componente de Balances
<Transfer
    api={api}
    keyring={keyring}
/>
Puedes obtener la versión de este código en el directorio part-3-1:
cd part-3-1;
yarn;
yarn start;
Si ejecutas este ejemplo, encontrarás nuestro componente Transfer debajo de la tabla de Balance.