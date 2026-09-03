# ADR-0002: Concentrar todo el hardware de barrera en la PCB Potencia

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

Decidido el aislamiento galvanico (ADR-0001), habia que elegir donde vive fisicamente la barrera. La PCB de Potencia es ademas la base mecanica sobre la que se acoplan las otras dos.

## Decision

**Todo el hardware de aislamiento se concentra en la PCB de Potencia**: drivers aislados, sensores de corriente, amplificador aislado de tension de bus, fuente aislada y el aislador digital del enlace con la E/S.

En particular, el **aislador digital del enlace con la PCB E/S se coloca en la Potencia**, no en la E/S.

## Alternativas consideradas

**Poner el aislador del enlace en la PCB E/S.** Se descarto porque obligaria a que el conector Potencia-E/S llevara a la vez señales frias y calientes, lo que exige separacion en el propio conector, y encareceria una placa de la que se van a hacer muchas variantes.

## Consecuencias

- Modelo mental muy nitido: **el Cerebro es 100 % frio, la E/S es 100 % caliente, y solo la Potencia se complica**.
- Las PCBs de E/S salen baratas y simples, que es justo lo que interesa porque son las que mas se van a rediseñar.
- El precio: toda PCB de Potencia lleva el aislador digital aunque no se use placa de E/S.
- Unica excepcion: el transceptor CAN aislado hacia el BMS vive en el Cerebro. No se considera realmente una excepcion, porque un bus que sale al exterior del equipo deberia ir aislado siempre.
