# ADR-0004: Señal de corriente normalizada y ratiometrica

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

El sensor elegido para el prototipo, la familia ACS725, se acaba en mas o menos 40 A. Para llegar a los 500 A previstos hay que cambiar de tecnologia por completo: sensores sin nucleo montados sobre un busbar (familia ACS3761x, de 100 A a mas de 4000 A) o sensores con nucleo integrado tipo LEM HST.

Si el Cerebro tuviera que saber que sensor lleva debajo, la modularidad se rompia.

## Decision

**Cada PCB de Potencia normaliza su propia señal de corriente**: salida ratiometrica con 0 A en VREF/2 y fondo de escala en la corriente nominal de esa placa. El Cerebro lee en tanto por uno y no sabe ni necesita saber que sensor hay debajo.

Ademas, **los sensores se alimentan del mismo LDO de precision que genera VREF+**, de modo que, siendo ambos ratiometricos, los errores de la referencia se cancelan.

## Alternativas consideradas

**Que el Cerebro conociera la sensibilidad de cada sensor en milivoltios por amperio.** Funciona, pero obliga a que la configuracion contenga parametros de bajo nivel especificos de cada sensor, en lugar de un unico numero con significado fisico claro.

## Consecuencias

- La placa de 500 A deja de ser un problema de arquitectura y pasa a ser solo un problema de diseño de potencia.
- Lo unico que la configuracion aporta es **cuantos amperios son el fondo de escala**.
- La cancelacion ratiometrica mejora la precision sin coste alguno.
- Queda un detalle de firmware asociado: el ACS725 tiene 2 microsegundos de retardo de propagacion y 4 de tiempo de respuesta, lo que a 16 kHz es alrededor del 6 % del periodo y desplaza el instante de muestreo respecto al centro del PWM. **Hay que compensarlo**, o aparecera como un rizado de par de origen desconocido. Los sensores sin nucleo, a 250 kHz, sufren mucho menos.
