# ADR-0013: FOC propio en lugar de X-CUBE-MCSDK

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

ST ofrece el X-CUBE-MCSDK, que genera un control de motor completo a partir de una configuracion grafica y acelera muchisimo el arranque de un proyecto de este tipo.

## Decision

Se desarrolla el **control FOC desde cero**, sin usar el X-CUBE-MCSDK.

## Alternativas consideradas

- **X-CUBE-MCSDK.** Mucho mas rapido de poner en marcha, pero genera codigo rigido y encajar dentro de su estructura la gestion de luces, el Bluetooth, el enlace con la PCB de E/S y la maquina de estados de seguridad produce bastante friccion.
- **Hibrido**: codigo propio usando el SDK y su herramienta de perfilado solo como referencia para caracterizar el motor. Sigue siendo una opcion util en la Fase 7 aunque el codigo sea propio.

## Consecuencias

- Control total sobre el lazo, la temporizacion del muestreo y la integracion con el resto del sistema.
- Mucho mas trabajo, concentrado en la Fase 5.
- Permite implementar limpiamente cosas que el SDK dificulta, como la **compensacion del retardo de propagacion del sensor de corriente** en el instante de muestreo (ver ADR-0004) y la calibracion de offset de cada canal con el puente apagado en cada arranque.
- La arquitectura del firmware podra separarse en capas de verdad: soporte de placa, control independiente del hardware, aplicacion de vehiculo y comunicaciones.
- El lazo de corriente ira siempre en una **interrupcion de maxima prioridad**, nunca dentro de una tarea de sistema operativo.
