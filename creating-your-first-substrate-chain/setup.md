# Setup
Normalmente le enseñaremos sobre el framework de desarrollo de blockchain Substrate, sin embargo, configurar tu equipo para desarrollar en Substrate puede tomar tiempo.

Para optimizar tu tiempo, tendremos que comenzar por el proceso de configuración. En la siguiente sección, mientras esté compilando, aprenderá más sobre Substrate y lo qué estamos "instalando".

## Prerrequisitos

Para desarrollar en Substrate, tu ordenador necesita algunos requisitos para establecer un entorno de trabajo para desarrollar.

Nota: Configurar tu equipo es probablemente la parte más complicada de este tutorial, por lo que no dejes que te desanime.

### Desarrollo en Substrate

Si estás usando una máquina basada en Unix (Linux, MacOS), hemos creado un "one-liner" o script de una línea para instalar todos los prerrequisitos por ti:

`curl https://getsubstrate.io -sSf | bash -s -- --fast` 

Note: Si no tenías instalado Rust antes de ejecutar este script, asegurate de añadir cargo a tu PATH o reiniciar tu terminal antes de continuar(la orden se dio en la última línea del script de salida).

Nota: Si quieres ver qué hace específicamente este script, visite: https://getsubstrate.io

Instalará automáticamente:

- CMake
- pkg-config
- OpenSSL
- Git
- Rust

Si usted está utilizando Windows y no dispone de Windows Subsystem for Linux, el proceso es un poco más complicado, pero está bien documentado aquí.

## Compilando Substrate

