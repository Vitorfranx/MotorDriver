# ADR-0010: Configuracion desde la app e identificacion por pines cableados

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

Se propuso que cada PCB de Potencia y de E/S llevara una memoria con su identidad (modelo, revision, numero de serie, rango del sensor de corriente, tension maxima, limites termicos), de forma que un unico firmware se autoconfigurase segun la placa conectada.

El usuario prefirio no poner memorias en cada placa y configurarlo todo desde la app movil.

## Decision

- **La configuracion se introduce desde la app** y se guarda en el Cerebro.
- La **PCB de Potencia se identifica con 4 pines del conector** unidos a masa o dejados al aire, formando un codigo de 16 familias. No es una memoria: es un **enclavamiento de seguridad**.
- La **PCB de E/S se identifica por autodescripcion** sobre el enlace UART (ver ADR-0007), sin ningun componente adicional.

## Alternativas consideradas

- **Memoria de identificacion en cada placa** (EEPROM o FRAM). Es la solucion mas completa y la que convertiria esto en una plataforma de verdad, pero añade un componente y un coste por placa que el usuario prefirio evitar.
- **Confiar solo en la configuracion de la app**, sin enclavamiento. Se descarto porque nada impediria que un Cerebro configurado para la placa de 48 V acabara enchufado a la de 144 V. **El primer pulso de PWM con la escala de corriente equivocada se lleva la placa por delante.**
- **Una sola resistencia leida por el ADC** en lugar de cuatro pines digitales. Daria muchos mas codigos con un solo componente. Queda como opcion si 16 familias se quedaran cortas.

## Consecuencias

- El enclavamiento cuesta **literalmente unas pistas de cobre**, ningun componente.
- El firmware debe **negarse a habilitar el puente** si el codigo cableado no coincide con el tipo de placa configurado en la app.
- La configuracion tiene que guardarse en algun sitio no volatil del Cerebro, lo que justifica la FRAM (ver ADR-0014).
- La app puede construir su interfaz dinamicamente a partir de la autodescripcion de la placa de E/S.
