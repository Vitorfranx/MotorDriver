# Plan de fases

El proyecto se desarrolla en sesiones de trabajo distintas. Este documento fija el orden. El estado real de avance esta en [`00-ESTADO.md`](00-ESTADO.md).

---

## Fase 0 — Repositorio y documentacion

**Completada.**

Creacion del repositorio y de la documentacion viva que congela la arquitectura acordada: requisitos, arquitectura de dominios, contrato entre placas, protocolo del enlace y registro de decisiones.

## Fase 1 — Disponibilidad y cierre de la lista de materiales

Comprobar, componente a componente, la disponibilidad real en la biblioteca de JLCPCB, anotando la referencia `Cxxxxxx` y una alternativa para cada uno.

Salidas de la fase:

- Los tres `hardware/*/bom.csv` rellenos con los componentes criticos verificados.
- Decision cerrada sobre el microcontrolador de la PCB E/S.
- Decision cerrada sobre el smart switch.
- Decision cerrada sobre el sensor de corriente (sensor integrado o shunt con amplificador aislado).
- Confirmacion de que el STM32G474QET6 se puede montar, o plan alternativo.

Es **la fase que mas puede cambiar el diseño**, y por eso va antes de dibujar nada.

## Fase 2 — Interfaces y configuracion del microcontrolador

- Fijar el pinout definitivo de los dos conectores entre placas en [`03-icd-conectores.md`](03-icd-conectores.md), y congelarlo como version 1.0 del ICD.
- Decidir el apilado mecanico y el tipo de conector.
- Configurar el proyecto en STM32CubeMX y generar el `.ioc`.
- Rellenar [`05-asignacion-pines.md`](05-asignacion-pines.md) de forma coherente con el `.ioc`.

## Fase 3 — Esquematicos

Los dibuja el usuario en su herramienta. El trabajo aqui es de **revision**: comprobar coherencia con el ICD, con la lista de materiales y con las reglas de aislamiento, y señalar errores.

Orden sugerido: primero Potencia (por ser la base y la que sostiene la barrera), luego Cerebro, luego E/S.

## Fase 4 — Firmware base

- Arranque, relojes y comprobaciones iniciales.
- Enlace con la PCB E/S y protocolo completo.
- Lectura de sensores: Hall, encoder, corrientes, tension de bus, temperaturas.
- Maquina de estados de seguridad y tabla declarativa de fallos.
- Calibracion de offset de los sensores de corriente con el puente apagado.
- **Sin mover el motor todavia.**

## Fase 5 — Control FOC

- Transformadas de Clarke y Park, reguladores de corriente, modulacion vectorial.
- Compensacion del retardo de propagacion del sensor de corriente en el instante de muestreo.
- Arranque con sensores Hall.
- Frenada regenerativa.
- Eventualmente, debilitamiento de campo y control sin sensores.

## Fase 6 — Conectividad

- Firmware del ESP32-C6, servicio BLE y protocolo de aplicacion.
- Cargador de arranque del STM32 con doble banco y vuelta atras.
- Cadena de actualizacion por aire, incluida la cascada hacia la PCB E/S.
- Aplicacion movil, con modo cuadro de instrumentos en horizontal.
- Dialogo por CAN con el BMS.

## Fase 7 — Pruebas en el vehiculo

- Caracterizacion del motor del patinete.
- Ajuste de reguladores.
- Pruebas de todos los modos de fallo, uno a uno.
- Rodaje real.
