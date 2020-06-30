# Formato de direcciones SS58

SS58 es un formato de direcciones simples diseñado para blockchains hechas en Substrate. No hay ningún problema con usar otros formatos de direcciones en la blockchain, pero éste formato es robust y es utilizado por defecto. Se basa fuertemente en el formato de comprobación de Bitcoin Base-58 con unas ligeras modificaciones.

La idea básica es un valor codificado en base-58 que puede ser identificar una determinada cuenta en la blockchain de substrate. Diferentes blockchain tienen distintas maneras de identificar cuentas. SS58 está diseñado para que sea ampliable o extensible por este motivo.

La actual especificación del formato SS-58 puede ser encontrada en la wiki de GitHub de Substrate:

https://github.com/paritytech/substrate/wiki/External-Address-Format-(SS58)