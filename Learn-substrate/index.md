# Resumen
Esta página es el punto de inicio a la documentación de Substrate.

Substrate es un framwork de desarrollo de blockchain con una Función de Transición de Estado (STF) genérica y componentes para consensos, red y configuración.

A pesar de ser "completamente genérico", dispone de estándares y convencionalismos - especialmente, con el Substrate runtime module library (También conocido como FRAME) - con respecto a las estructuras de datos que recibe el STF (State Transition Function), hace realidad el rápido desarrollo de blockchain.

## Uso
Substrate está diseñado para ser utilizado de alguna de las 3 formas siguientes:

- 1.Con el nodo Substrate: Puede ejecutar el nodo Substrate prediseñado y configurarlo con un bloque de génesis que incluye el tiempo de ejecución predeterminado del nodo. En este caso, simplemente, configure un archivo JSON y ponga en marcha su propia Blockchain. Ésto permite muy poca personalización, solamente se podrían modificar parámetros del génesis, los cuáles están incluidos en el módulo runtime node, estos parámetros serían; balances, staking, periodo de bloques, fees y governance... Para hacer un tutorial sobre ésto, vea: Iniciar una red privada con Substrate.
-
-