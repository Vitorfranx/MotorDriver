# Estado del proyecto

> Este es el documento de entrada. Actualizalo al final de cada sesion de trabajo.

**Ultima actualizacion**: 3 de septiembre de 2026
**Fase actual**: Fase 0 completada. Siguiente: Fase 1 (verificacion de disponibilidad en JLCPCB).

---

## Que esta hecho

- Arquitectura del sistema definida y cerrada: tres PCBs, dominios electricos, barrera de aislamiento y contrato entre placas. Ver [`02-arquitectura.md`](02-arquitectura.md).
- Requisitos del primer prototipo (patinete de 48 V) recogidos en [`01-requisitos.md`](01-requisitos.md).
- Protocolo del enlace Cerebro-E/S especificado en [`04-protocolo-es.md`](04-protocolo-es.md).
- Quince decisiones de arquitectura registradas en [`decisiones/`](decisiones/).
- Repositorio creado y publicado en `https://github.com/Vitorfranx/MotorDriver` (privado).

## Que queda pendiente

Ver el detalle de fases en [`06-plan-de-fases.md`](06-plan-de-fases.md).

Lo inmediato es la **Fase 1**: comprobar en la biblioteca de JLCPCB la disponibilidad real de los componentes criticos y cerrar la lista de materiales, anotando la referencia `Cxxxxxx` y una alternativa para cada uno.

Componentes criticos a verificar, por orden de riesgo:

| Componente | Riesgo | Plan B previsto |
|---|---|---|
| Smart switch SPOC+2 (BTS72220-4ESA) | Alto, es pieza de automocion | PROFET discretos, o equivalente del catalogo de JLCPCB |
| Sensor de corriente ACS725 de mas/menos 30 A | Alto, es pieza de automocion | Shunt mas amplificador aislado (NSI1200 o similar) |
| STM32G474QET6 en LQFP128 | Medio, figura en la biblioteca (C730129) pero puede requerir pedido anticipado | Pre-order o global sourcing de JLCPCB; soldadura manual; o bajar a LQFP100 |
| Modulo ESP32-C6 con conector U.FL | Medio | Otro modulo precertificado con U.FL |
| Bornes push-in para sensores | Medio, JLCPCB monta pocos conectores | Soldadura manual por el usuario |
| Driver aislado NSI6602 | Bajo, hay stock confirmado en LCSC | UCC21520 o equivalente |
| Amplificador aislado NSI1311 | Bajo | AMC1311 o ACPL-C87x |
| FRAM FM24CL64B-G | Bajo | MB85RC256V, o EEPROM con emulacion en flash |

## Decisiones abiertas

Estas cuestiones **no** estan cerradas y hay que resolverlas antes de avanzar a las fases que dependen de ellas:

1. **Microcontrolador de la PCB E/S**: STM32C071, STM32C031 o STM32G0B1. Criterio: disponibilidad en JLCPCB primero, encaje tecnico despues. El STM32C071 es el favorito por tener todos sus GPIO tolerantes a 5 V.
2. **Modelo concreto de smart switch** de la PCB E/S, dependiente de la disponibilidad.
3. **Modelo concreto de MOSFET** de la PCB Potencia y dimensionado del disipador.
4. **Topologia de las fuentes**: flyback con dos secundarios frente a buck caliente mas secundario aislado independiente.
5. **Frecuencia de PWM y del lazo de corriente**, aun sin fijar.
6. **FreeRTOS frente a superbucle** para la capa de aplicacion. El lazo de corriente ira siempre en interrupcion, eso no esta en discusion.
7. **Alcance del STO**: si es un requisito normativo segun IEC 61800-5-2, con doble canal y diagnostico, o simplemente una entrada funcional que corta el par.
8. **Control sensorless**: si se implementa, y si seria como respaldo o para alta velocidad.
9. **Apilado mecanico y conectores concretos** entre placas. Propuesta preferida: la Potencia como base grande atornillada al disipador, con Cerebro y E/S montados encima uno al lado del otro. Pendiente de la Fase 2.
10. **Protocolo CAN del BMS**: que BMS concreto y que mensajes expone.
11. **Tecnologia de la app movil** y plataformas objetivo. Entra en el alcance, pero cuando las PCBs esten hechas.

## Verificaciones pendientes en el vehiculo real

- **Confirmar si los interruptores del manillar comparten masa con el chasis.** Se ha asumido el caso peor (que si la comparten), y por eso toda la E/S del mazo vive en el dominio caliente. Si resultara que tienen retorno propio, no habria que cambiar nada, solo se ganaria margen.
- Caracterizar el motor del patinete: numero de pares de polos, resistencia e inductancia de fase, constante de par, y secuencia de los sensores Hall.
