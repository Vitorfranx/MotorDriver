# ADR-0011: Encoder con transceptores RS-485 y conector unificado

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

El Cerebro debe soportar encoder incremental TTL diferencial, BiSS-C y SSI. Se planteo si servirian amplificadores diferenciales genericos, un modulo RS-232 o un modulo RS-485.

Aclaracion de terminologia, porque era el origen de la duda:

- **RS-232** es asimetrica, con niveles de mas o menos 3 a 15 V, punto a punto y lenta. **No sirve** por ninguna de las tres cosas.
- **RS-422** es diferencial, punto a punto y de decenas de Mbit/s. Es **literalmente el estandar** que usan los encoders TTL diferenciales y el BiSS-C. Cuando un fabricante dice "salida TTL con line driver", esta diciendo RS-422.
- **RS-485** comparte la misma capa electrica que RS-422; la diferencia es que es semiduplex multipunto, con el emisor en triestado.

## Decision

**Tres transceptores RS-485**, uno por par diferencial, con habilitaciones independientes controladas por firmware, y **un unico borne push-in de 8 posiciones** (6 señales, 5 V y masa) que sirve para los tres interfaces:

| Modo | Configuracion |
|---|---|
| Incremental TTL | Los tres en recepcion: A, B y Z |
| BiSS-C | Par 1 en emision (reloj), par 2 en recepcion (datos) |
| SSI | Igual que BiSS-C |

El BiSS-C se lee con un **SPI en modo maestro usando solo SCK y MISO**: el reloj hace de MA, la entrada recoge SLO, se sacan mas bytes de reloj de los necesarios por DMA, y luego por software se busca el bit de inicio, se realinea el dato y se comprueba el CRC. Ese sobremuestreo es necesario porque el numero de bits que tarda el encoder en empezar a responder es variable.

## Alternativas consideradas

- **Amplificadores diferenciales genericos.** Funcionarian, pero seria construir a mano un comparador con histeresis cuando un receptor de linea ya trae los umbrales de mas o menos 200 mV, la polarizacion de seguridad y la proteccion adecuada.
- **RS-232.** No sirve.
- **Un conector por interfaz.** Triplicaria los bornes en el borde de la placa sin ganar nada.

## Consecuencias

- El firmware selecciona el modo con las habilitaciones, **sin ningun puente ni jumper**.
- Requisitos obligatorios del transceptor: **true failsafe**, para que un encoder desconectado de un nivel estable en lugar de un tren de pulsos fantasma; y velocidad suficiente para los 10 MHz que puede pedir el BiSS-C.
- El rango de modo comun de un transceptor RS-485 (de menos 7 a mas 12 V) permite leer sin problema un encoder alimentado a 5 V aunque el transceptor vaya a 3,3 V. En sentido contrario, un reloj emitido a 3,3 V cumple de sobra los minimos de RS-422.
- Terminacion fija de 120 ohmios por par, comun a todos los modos.
- Compromiso a decidir en la Fase 3: los transceptores rapidos tienen flancos rapidos y por tanto mas emisiones radiadas. Existen variantes con pendiente limitada, pero no llegan a la velocidad maxima del BiSS-C.
- El DMA del encoder debe tener **prioridad superior** al del enlace con la E/S, para que la lectura de angulo no sufra jitter.
- Los sensores Hall van por su propio camino: son tres señales asimetricas que necesitan adaptacion a 3,3 V y pines tolerantes a 5 V.
