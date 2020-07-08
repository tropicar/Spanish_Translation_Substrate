# Criptografía

Este documento ofrece un resumen conceptual de la criptografía utilizada en Substrate.

## Algoritmos de hash

Las funciones de hash son utilizadas en Substrate para mapear datos de tamaño arbitrario a valores de tamaño fijo.

Substrate proporciona dos algoritmos de hash de serie, pero es compatible con cualquier algoritmo que implemente el trait [hasher][hasher]

### xxHash

xxHash es una función rápida de hash [no-criptográfica][no-criptográfica], trabaja a velocidades cercanas a los límites de la RAM. Porque xxHash no es criptográficamente securo, es posible que la salida del algoritmo de hash puede ser razonablemente controlada modificando la entrada.Ésto permite a un usuario atacar este algoritmo haciendo colisiones de claves, colisiones de hash y árboles de almacenamientos desequilibrados.

xxHash es utilizado dónde la parte externa no puede manipular la entrada de la función hash. Por ejemplo, es utilizado para generar la clave para almacenar los valores del runtime, cuyas entradas son son controladas por el runtime developer.

Substrate utiliza la implementación [twox-hash][twox-hash] en Rust.

[hasher]: https://crates.parity.io/sp_core/trait
[no-criptográfica]: https://en.wikipedia.org/wiki/Hash_function
[twox-hash]: https://github.com/shepmaster/twox-hash

### Blake2

[Blake2][Blake2] es una función criptográfica de hash. Creado para que fuese muy rápido y que fuera utilizado en [Zcash][Zcash].

Substrate utiliza la implementación de [Blake-2][Blake-2].

## Criptografía de Clave Pública

La criptografía de clave pública es utilizada en Substrate para proporcionar un sistema de autenticación robusto.

Substrate proporciona diferentes esquemas cristográficas y es genérico de tal manera que es compatible con cualquier implementación que tenga el trait de [Pair][Pair].

### ECDSA 

Substrate proporciona un esquema de firmas [ECDSA][ECDSA]cusando la curva [secp256k1][secp256k1]. Este es el mismo algoritmo criptográfico utilizado para securizar [Bitcoin][Bitcoin] y [Ethereum][Ethereum].

### Ed25519

[Ed25519][Ed25519] es un esquema de firma EdDSA usando [Curve25519][Curve25519]. Está cuidadosamente diseñado en varios niveles para lograr altas velocidades sin comprometer seguridad.

### SR25519

[SR25519][SR25519] está basado en la misma curva que Ed25519. Sin embargo, utiliza firmas de Schnorr en lugar del esquema de
EdDSA.

Las firmas Schnorr traen algunas características destacables sobre los esquemas:

- 
- 
- 

Un sacrificio que se hace al usar firmas Schnorr sobre ECDSA es que ambos requieren 64 bytes, pero sólo las firmas ECDSA comunican su clave pública.

## Siguientes Pasos

### Aprende más

- Aprende acerca de las abstracción de cuentas en Substrate.
- Para las descripciones detalladas sobre la criptografía utilizada por favor consulte la mas avanzada [wiki][wiki].

### Ejemplos

- Mira el módulo de peticiones de Polkadot para ver como [verificar-las-firmas-de-Ethereum-en-el-runtime-de-Substrate][verificar-las-firmas-de-Ethereum-en-el-runtime-de-Substrate].



[Blake2]: https://en.wikipedia.org/wiki/BLAKE_(hash_function)#BLAKE2

[Zcash][Zcash]: https://en.wikipedia.org/wiki/Zcash

[Blake-2]: https://docs.rs/blake2/

[Pair]: https://crates.parity.io/sp_core/crypto/trait

[ECDSA]: https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm

[secp256k1]: https://en.bitcoin.it/wiki/Secp256k1

[Bitcoin]: https://en.wikipedia.org/wiki/Bitcoin

[Ethereum]: https://en.wikipedia.org/wiki/Ethereum

[Ed25519]: https://en.wikipedia.org/wiki/EdDSA#Ed25519

[Curve25519]: https://en.wikipedia.org/wiki/Curve25519

[SR25519]: https://research.web3.foundation/en/latest/polkadot/keys/1-accounts-more.html

[wiki]: https://research.web3.foundation

[verificar-las-firmas-de-Ethereum-en-el-runtime-de-Substrate]: https://github.com/paritytech/polkadot/blob/master/runtime/common/src/claims.rs