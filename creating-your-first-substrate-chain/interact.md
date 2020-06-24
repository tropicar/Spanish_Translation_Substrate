# Interactuando con tu nodo

Ahora que su nodo ha terminado de compilar, vamos a mostrarle cómo funciona todo.

## Iniciando tu nodo

Ejecuta los siguientes comandos para inicializar su nodo:
`cd substrate-node-template/`
`#purge-chain elimina los resto de datos antiguos después de ejecutar un nodo en modo 'dev' en el pasado`
`#Te pedirá que escribras 'y'o 's'`
`./target/release/node-template purge-chain --dev`
`#Ejecuta tu nodo en modo "developer"` 
`./target/release/node-template --dev`
Debería ver algo parecido a ésto si su nodo está ejecutándose correctamente:
`./target/release/node-template --dev`

~~~
2019-09-05 15:57:27 Running in --dev mode, RPC CORS has been disabled.
2019-09-05 15:57:27 Substrate Node
2019-09-05 15:57:27   version 2.0.0-b6bfc95-x86_64-macos
2019-09-05 15:57:27   by Anonymous, 2017, 2018
2019-09-05 15:57:27 Chain specification: Development
2019-09-05 15:57:27 Node name: unwieldy-skate-4685
2019-09-05 15:57:27 Roles: AUTHORITY
2019-09-05 15:57:27 Initializing Genesis block/state (state: 0x26bd…7093, header-hash: 0xbf06…58a9)
...
2019-09-05 15:57:30 Imported #1 (0x9f41…e673)
2019-09-05 15:57:32 Idle (0 peers), best: #1 (0x9f41…e673), finalized #1 (0x9f41…e673), ⬇ 0 ⬆ 0
2019-09-05 15:57:37 Idle (0 peers), best: #1 (0x9f41…e673), finalized #1 (0x9f41…e673), ⬇ 0 ⬆ 0
~~~ 

Si el número después de best: va incrementado, ¡tu blockchain está produciendo nuevos bloques!

## Iniciar el Front End

Para interactuar con el nodo local, usaremos la interfaz de usuario Polkadot-js Apps, también conocida como "Apps". A pesar del nombre, Apps funcionará con cualquier blockchain basadaen Substrate, incluida la nuestra, no solo Polkadot.

En su navegador, diríjase a:
https://polkadot.js.org/apps/

Algunos navegadores, especialmente Firefox, no se conectarán a un nodo local desde una web https. Una solución fácil es probar otro navegador, como Chromium. Otra opción es alojar esta interfaz localmente.

## Interactuar

Seleccione la pestaña Accounts, verá las cuentas de prueba a las que tiene acceso. Algunas, como Alice y Bob, ya tienen fondos!

Puede probar a transferir algunos fondos desde Alicia a Charlie haciendo click en el botón "send".

Si todo va correctamente, deberían aparecerle varias notificaciones avisándole sobre "Extrinsic Success", y por supuesto, el balance de Charlie incrementará.

## Siguientes pasos

Este es el final de tu viaje para poner en marcha tu primer blockchain con Substrate.

Has puesto en marcha una blockchain funcional basada en Substrate, con una criptomoneda, ha adjuntado una interfaz de usuario a esa cadena, y ha hecho transferencias entre los usuarios. Esperamos que sigas aprendiendo sobre Substrate.

Sus siguientes pasos podrían ser:
- Descentraliza tu red con más nodos en el tutorial Inicia una red privada.
- Añadir funcionalidad personalizada en el tutorial Construir un dApp.
Si encuentras algún problema con este tutorial o quiere mostrarnos sugerencías u apoyarnos, tómate la libertad de abrir una issue de Github con tus ideas.