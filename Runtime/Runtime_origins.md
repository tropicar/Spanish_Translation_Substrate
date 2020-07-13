# Runtime Origins (Posibles orígenes en el Runtime)

El runtime origin es utilizado por las funciones implementadas para comprobar de dónde procede la llamada a dicha función.

## Raw Origins

Substrate define tres raw origins que pueden ser utilizados en tu módulo del runtime:

~~~
pub enum RawOrigin<AccountId> {
    Root,
    Signed(AccountId),
    None,
}
~~~

- Root: Origen a nivel de sistema (system level origin). Éste es el nivel de privilegios más alto.

- Signed (firmado): Proveniente de una transacción (A transaction origin). Ésto está firmado por una clave pública e incluye 
el identificador de la cuenta del que ha firmado.

- Ninguno (None): Carece de origen. Éste debe ser acordado entre los validadores o validado por un módulo para ser incluido.

## Custom Origins (Orígenes personalizados)

También puedes definir orígenes personalizados, que pueden ser utilizados para comprobar la autenticación en las funciones del runtime.

Más detalles por hacer

## Custom Origin Call (Llamada con un origen personalizado)

Puedes construir llamadas dentro de tu runtime sin ningún origen. Por ejemplo:

~~~
// Root
proposal.dispatch(system::RawOrigin::Root.into())

// Signed
proposal.dispatch(system::RawOrigin::Signed(who).into())

// None
proposal.dispatch(system::RawOrigin::None.into())
~~~

Puedes ver el código fuennte del módulo Sudo como una implementación práctica de ésto.

## Siguientes Pasos
### Aprende Más

- Learn origin is used in the decl_module macro.
- Aprender (Por hacer)

### Ejemplos

- Ver el módulo Sudo para ver como permite a un usuario llamar con Root y origen firmado o Signed.

- Ver el módulo Timestamp para ver cómo valida una llamada sin origen (Noneorigin).

- Ver el módulo Collective para ver cómo crear un miembro de origen personalizado (customMemberorigin).

- Vea nuestra guía para crear y utilizar un custom origin (origen personalizado).

### Referencias

Visita la documentación del trait RawOrigin.