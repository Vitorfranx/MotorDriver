# Documentacion de referencia

Datasheets y manuales de los componentes del proyecto.

## Indice

| Fichero | Contenido |
|---|---|
| `DS_stm32g474qe.pdf` | Hoja de caracteristicas del STM32G474xE. Patillaje, caracteristicas electricas, tolerancia a 5 V de cada pin y funciones alternativas. Es el documento a consultar al asignar pines (Fase 2). |
| `rm0440-stm32g4-series-advanced-armbased-32bit-mcus-stmicroelectronics.pdf` | Manual de referencia RM0440 de la serie STM32G4. Descripcion funcional de todos los perifericos: temporizadores avanzados, HRTIM, ADC, CORDIC, FDCAN y organizacion de la flash de doble banco. |

## Documentos que conviene añadir

A medida que se cierren los componentes en la Fase 1:

- Driver de puerta aislado seleccionado.
- Sensor de corriente seleccionado.
- Amplificador aislado de tension de bus.
- Aislador digital del enlace con la PCB E/S.
- Microcontrolador de la PCB E/S.
- Smart switch seleccionado.
- Modulo ESP32-C6.
- Transceptores RS-485 del encoder.
- Manual del encoder que se instale en el motor, con su protocolo concreto.

> Estos PDF se versionan en el repositorio a proposito: tener la documentacion junto al diseño evita trabajar con revisiones distintas del mismo documento en sesiones distintas.
