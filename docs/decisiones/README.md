# Registro de decisiones de arquitectura (ADR)

Cada fichero de esta carpeta recoge **una decision** y, sobre todo, **por que** se tomo. El valor de este registro no esta en la decision, que ya se ve en el diseño, sino en el razonamiento y en las alternativas que se descartaron: es lo que evita volver meses despues a discutir algo que ya estaba resuelto.

## Reglas

- Una decision revocada **no se borra**: se marca como sustituida y se crea un ADR nuevo que la reemplace, indicando cual sustituye a cual.
- No se revoca un ADR sin avisar explicitamente al usuario.
- Los ADR se numeran de forma correlativa y no se reutilizan numeros.

## Plantilla

```markdown
# ADR-XXXX: Titulo

**Fecha**: dd de mes de aaaa
**Estado**: Aceptada | Sustituida por ADR-YYYY | Obsoleta

## Contexto

Que problema habia y que restricciones existian.

## Decision

Que se decidio, en una o dos frases.

## Alternativas consideradas

Que otras opciones habia y por que se descartaron.

## Consecuencias

Que implica esta decision, tanto lo bueno como el precio que hay que pagar.
```

## Indice

| ADR | Titulo | Estado |
|---|---|---|
| [0001](ADR-0001-aislamiento-galvanico.md) | Aislamiento galvanico real entre control y potencia | Aceptada |
| [0002](ADR-0002-barrera-en-potencia.md) | Concentrar todo el hardware de barrera en la PCB Potencia | Aceptada |
| [0003](ADR-0003-drivers-aislados.md) | Drivers de puerta aislados como estandar de plataforma | Aceptada |
| [0004](ADR-0004-corriente-normalizada.md) | Señal de corriente normalizada y ratiometrica | Aceptada |
| [0005](ADR-0005-tension-bus-aislada.md) | Tension de bus por amplificador aislado con escala normalizada | Aceptada |
| [0006](ADR-0006-ntc-aisladas.md) | NTC aisladas fisicamente y leidas directamente | Aceptada |
| [0007](ADR-0007-mcu-en-es.md) | Microcontrolador dedicado en la PCB E/S | Aceptada |
| [0008](ADR-0008-enlace-uart.md) | Enlace Cerebro-E/S por UART aislado | Aceptada |
| [0009](ADR-0009-esp32c6-en-cerebro.md) | ESP32-C6 en el Cerebro para Bluetooth y actualizacion por aire | Aceptada |
| [0010](ADR-0010-configuracion-desde-app.md) | Configuracion desde la app e identificacion por pines cableados | Aceptada |
| [0011](ADR-0011-encoder-rs485.md) | Encoder con transceptores RS-485 y conector unificado | Aceptada |
| [0012](ADR-0012-descartar-1vpp.md) | Descartar el encoder de 1 Vpp en la revision 1 | Aceptada |
| [0013](ADR-0013-foc-propio.md) | FOC propio en lugar de X-CUBE-MCSDK | Aceptada |
| [0014](ADR-0014-fram.md) | FRAM para parametros y registro de vida | Aceptada |
| [0015](ADR-0015-bom-jlcpcb.md) | Restriccion de la lista de materiales a la biblioteca de JLCPCB | Aceptada |
