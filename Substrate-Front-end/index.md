# Introducción 

Ésta es una guía de nivel principiante para iniciarse en la creación del front-end de una aplicación descentralizada (más adelante llamaremos a estas aplicaciones DApp) basada en una blockchain de Substrate usando la API de Polkadot-js. Este tutorial no requiere de ningún conocimiento previo en el framework de Substrate o del lenguaje Rust. Sin embargo, requiere conocimientos de JavaScript así como conocimiento básico del framework React.
¿Qué aspectos cubre esta guía?
Este tutorial es para desarrolladores de front-end que quieran ser guíados en el uso de la API de Polkadot-js y las buenas prácticas en el desarrollo de una interfaz de usuario en una blockchain basada en Substrate. El objetivo es crear una aplicación que permita mostrar las cuentas de los usuarios, ver sus balances, y ver los fondos entre las diferentes cuentas. El código para esta aplicación está disponible en el repositorio substrate-front-end.
SI encuentras algún problema durante la realización de este tutorial o algo que no funcione como es debido, por favor contáctanos en Riot, envía una issue/PR a este documento haciéndo click en Edit en la esquina derecha de cualquiera artículo, o sube una issue/PR en el repositorio de substrate-front-end.
La guía se divide en las siguientes partes:
Parte 1:Conéctate a un nodo y muestra la información básica
Framework y herramientas
Conectarse a un nodo
Mostrar información del nodo
PART 2:Peticiones (queries) y mostrado de balances
Obtener cuentas de prueba (testing accounts)
Consultar y mostrar el balance de Alice
Consultar y mostrar todos los balances
Conceptos importantes
PARTE 3:Transferir fondos
Transferir fondos (o tokens) entre cuentas de usuario
Conceptos importantes
PARTE 4:Uso de cuentas externas
Instalar una extensión para administrar cuentas
Mostrar cuentas "externally-injected"
Adaptar nuestro componente Transfer
Conceptos importantes
Más allá: Extraer el botón de enviar dentro de su propio componente