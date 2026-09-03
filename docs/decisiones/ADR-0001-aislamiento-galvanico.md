# ADR-0001: Aislamiento galvanico real entre control y potencia

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

El primer prototipo es de 48 V, es decir, por debajo de los 60 V de corriente continua, asi que no hay riesgo electrico para las personas y el aislamiento no es un requisito de seguridad. La mayoria de los drivers de patinete usan masa comun, con el negativo de bateria como referencia de todo el sistema.

Sin embargo, la arquitectura debe escalar hasta 144 V y 500 A reutilizando la misma PCB de control.

## Decision

Se adopta **aislamiento galvanico real** entre el dominio de control y el de potencia desde el primer prototipo.

## Alternativas consideradas

**Masa comun**, con el negativo de bateria como referencia unica. Es lo habitual en vehiculos ligeros, mas barato y mas simple. Se descarto porque obligaria a rediseñar la PCB Cerebro al subir de tension, que es exactamente lo que la arquitectura modular pretende evitar.

## Consecuencias

- La PCB Cerebro queda **agnostica a la tension del vehiculo**: la misma placa sirve para una PCB de Potencia de 48 V y para una de 144 V.
- Mejora la inmunidad al ruido del inversor.
- Encarece el diseño: drivers aislados, fuente aislada, amplificador aislado para la tension de bus y aislador digital para el enlace con la E/S.
- **Obliga a que toda la E/S del mazo del vehiculo viva en el dominio caliente**, porque las luces y los interruptores retornan por el chasis. Esta es la consecuencia menos evidente y la que mas condiciona el reparto de responsabilidades entre placas.
- El montaje mecanico del Cerebro debe usar separadores aislantes, o el aislamiento se pierde por los tornillos.
- Al depurar hay que tener cuidado: con el Cerebro flotante, el programador arrastra la masa del PC al lado frio. Conectar a la vez un osciloscopio al lado de potencia cortocircuitaria la barrera.
