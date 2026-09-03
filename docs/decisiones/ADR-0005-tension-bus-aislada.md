# ADR-0005: Tension de bus por amplificador aislado con escala normalizada

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

El divisor resistivo que mide la tension de bus esta necesariamente en el lado caliente, asi que la señal tiene que cruzar la barrera. Se descarto expresamente hacerlo por un bus serie: el usuario prefirio lectura analogica directa.

## Decision

Usar un **amplificador aislado** de entrada de alta impedancia y rango de 0 a 2 V, cuya salida entra directamente al ADC del Cerebro ya en el lado frio.

Ademas, **el divisor de cada PCB de Potencia se dimensiona para que 2,0 V de salida correspondan a la tension maxima nominal de esa placa**. El Cerebro lee en tanto por uno.

Candidatos: AMC1311 de Texas Instruments o NSI1311 de Novosense, que es equivalente y esta mucho mejor surtido en LCSC.

## Alternativas consideradas

- **Un ADC serie en el lado caliente**, colgado del mismo bus aislado del enlace con la E/S. Habria colapsado tension de bus, NTC e identificacion en un solo cruce, pero se descarto por preferencia del usuario de leer directamente.
- **Modulador delta-sigma aislado** con salida digital. Se descarto porque el STM32G4 no tiene filtro digital dedicado y habria que hacer el filtro sinc por software.
- **Optoacoplador lineal.** Mas barato, pero exige calibracion y tiene deriva.

## Consecuencias

- Necesita una pequeña alimentacion en el lado caliente, que sale del rail de 12 V que la placa ya tiene para los drivers.
- El amplificador detecta si le falta la alimentacion del lado caliente, lo que es un diagnostico util y gratuito.
- Con 1500 V eficaces de tension de trabajo, los 144 V previstos quedan con un margen enorme.
- Igual que con la corriente, **la configuracion solo aporta un numero**: la tension maxima de la placa.
