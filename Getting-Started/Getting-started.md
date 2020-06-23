# Getting started

Esta página tiene todo lo que necesitarás para empezar a construir con Substrate.
Obtener Ayuda
Substrate es un framework innovador para el desarrollo de Blockchain, por ello, es posible que sufra ligeras modificaciones y algunos errores.
Os hacemos saber que nos hace muy feliz el poder ayudaros en la iniciación con Substrate, por lo que si encontráis algún error, avisadnos mediante:
Abrir una issue en Github
Háblanos en el canal de Riot
Prerequisitos
Para desarrollar en el ecosistema de Substrate, debes configurar tu entorno de desarrollo. Dependiendo del sistema operativo que utilices, las instrucciones que necesites puedes ser diferentes.
Instalación rápida
Sistemas operativos basados en Unix
Mac OS, Arch, o un sistema operativo basado en Debian como Ubuntu:
curl https://getsubstrate.io -sSf | bash
Instalación manual
Debian
Ejecuta:
sudo apt install -y cmake pkg-config libssl-dev git gcc build-essential clang libclang-dev
MacOS
Instala el gestor de paquetes Homebrew, luego ejecuta:
brew install openssl cmake llvm
Windows
Debido a que las instrucciones de instalación de Windows se diferencian mucho de los sistemas operativos estándar basados en Unix, hemos separado por secciones las distintas instrucciones de configuración en la parte inferior de esta página.
Ir a: Getting Started on Windows
Entorno de desarrollo para Rust
Substrate utiliza el lenguaje de programación Rust. Tendrás que instalar Rust usando rustup:
curl https://sh.rustup.rs -sSf | sh
A continuación, asegúrese de que está utilizando la última versión estable de Rust por defecto:
rustup default stable
Wasm Compilation
Substrate utiliza WebAssembly (Wasm), y necesitarás configurar tu compilador de Rust para utilizar nightly, el cuál permite usar wasm.
Ejecutar lo siguiente:
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
Actualización de Rustup
Substrate siempre utiliza la última versión estable de Rust y nightly para compilar. Para comprobar que tu compilador de Rust está siempre actualizado, tendrás que ejecutar:
rustup update
Ésto puede incluso solucionar algunos problemas que estuvieras teniendo al trabajar con Substrate.
Entorno de desarrollador front-End
Substrate usa Yarn para cubrir todas las necesidades del desarrollo en Front-end. Visita las instrucciones de instalación de Yarn para configurarlo en tu ordenador.
Asegúrese de que tiene la última versión estable de Node.js (>= 10. 6.3) y Yarn (>= 1.19.0).
Obtener código fuente
Dependiendo de lo que estés buscando crear, existen diferentes puntos de partida para el desarrollo en Substrate.
Nota: Puede añadir la opción --help a cualquier comando para ver opciones adicionales.
Substrate
El proyecto principal de Substrate contiene todas las librerías principales que utilizan las blockchain basadas en Substrate, un nodo predeterminado para ejecutar la testnet de Substrate, y una herramienta de generación de claves llamada Subkey.
Este punto de partida tiene sentido si planea contribuir al proyecto de Substrate o si desea montar una testnet de substrate o Subkey.
Obtener el proyecto:
git clone https://github.com/paritytech/substrate
Compila el proyecto con:
cargo build --release
Testear el proyecto con:
cargo test --all
Iniciar el nodo Substrate con:
./target/release/substrate
Instalar el nodo Substrate localmente
Puede instalar el binario del nodo Substrate localmente para poder poner en marcha un nodo fácilmente.
En la carpeta de proyecto Substrate, ejecutar:
cargo install --force --path ./bin/node/cli/
Puede ejecutar este binario generado con:
substrate
Instalar Subkey Localmente
Puede instalar el binario de Subkey para tener un fácil acceso a esta herramienta.
En la carpeta del proyecto Substrate, ejecutar:
cargo install --force --path ./bin/utils/subkey subkey
Ahora, puedes ejecutar el binario generado con:
subkey
Interactúa con Substrate
La forma más rápida de interactuar con cualquier red local o pública de Substrate es visitar Polkadot-JS Apps:

https://polkadot.js.org/apps/
Puedes encontrar la documentación del ecosistema de Polkadot-JS en:

https://polkadot.js.org/
Plantilla de nodos Substrate
Proporcionamos un nodo básico de Substrate destinado al desarrollo de nuevas blockchains de Substrate.
Obtener el proyecto en:
https://github.com/substrate-developer-hub/substrate-node-template
Compila el proyecto con:
cargo build --release
Purga cualquier nodo de desarrollo (existing developer node) con:
# Se te pedirá que escribas `y` o `s` para eliminar la base de datos
./target/release/node-template purge-chain --dev
Iniciar un nodo de desarrollador con:
./target/release/node-template --dev
Plantilla de Substrate Front-End
Proporcionamos un front-end básico de Substrate, counstruido con REACJS y la API de Polkdadot-JS. Este proyecto está diseñado para el desarrollo rápido y fácil de interfaces de usuario personalizadas.
Obtener el proyecto en:
https://github.com/substrate-developer-hub/substrate-front-end-template
Instalar las dependencias del nodo con:
yarn install
Ejecutar el front con:
yarn start
Conéctate al front-end en localhost:8000.
Nota: Necesita tener un nodo Substrate local ejecutándose para interactuar con esta interfaz de usuario.
Comenzando en Windows 10
Si está intentando configurar un equipo que utiliza Windows para usar Substrate, haga lo siguiente:
Descarga e instala "Build Tools for Visual Studio."
Puedes obtenerlo en este enlace: https://aka.ms/buildtools.
Ejecute el archivo de instalación: vs_buildtools.exe.
Asegúrese de que el componente "Windows 10 SDK" está incluido al instalar Visual C++ Build Tools.
Reinicie el ordenador.
Instalar Rust:
Las instrucciones detalladas son proporcionadas por el Libro de Rust.
Descargar de: https://www.rust-lang.org/tools/install.
Ejecute el archivo de instalación: vs_buildtools.exe.
Tenga en cuenta que no debería pedirle que instale vs_buildtools ya que lo hizo en el paso 1.
Elija "Default Installation"
Para empezar, necesita el directorio bin de Cargo (%USERPROFILE%\.cargo\bin) en su variable de entorno PATH. Las futuras aplicaciones tendrán automáticamente el entorno correcto, pero puede que necesite reiniciar el intérprete de comandos actual.
Ejecute los siguientes comandos en la consola (CMD) para configurar el entorno de Wasm:
    rustup update nightly
    rustup update stable
    rustup target add wasm32-unknown-unknown --toolchain nightly
Instala wasm-gc, es utilizado para reducir el tamaño de los archivos Wasm:
    cargo install --git https://github.com/alexcrichton/wasm-gc --force
Instalar LLVM: https://releases.llvm.org/download.html
Instalar OpenSSL con vcpkg:
    mkdir C:\Tools
    cd C:\Tools
    git clone https://github.com/Microsoft/vcpkg.git
    cd vcpkg
    .\bootstrap-vcpkg.bat
    .\vcpkg.exe install openssl:x64-windows-static
Añade OpenSSL a las variables del sistema usando PowerShell:
    $env:OPENSSL_DIR = 'C:\Tools\vcpkg\installed\x64-windows-static'
    $env:OPENSSL_STATIC = 'Yes'
    [System.Environment]::SetEnvironmentVariable('OPENSSL_DIR', $env:OPENSSL_DIR, [System.EnvironmentVariableTarget]::User)
    [System.Environment]::SetEnvironmentVariable('OPENSSL_STATIC', $env:OPENSSL_STATIC, [System.EnvironmentVariableTarget]::User)
Finalmente, instale cmake: https://cmake.org/download/
Ahora puedes volver a Obtener el código fuente para aprender cómo descargar y compilar Substrate!
