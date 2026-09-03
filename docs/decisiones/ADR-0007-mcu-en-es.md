# ADR-0007: Microcontrolador dedicado en la PCB E/S

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

Al asumir que los interruptores del manillar comparten masa con el chasis, toda la E/S del mazo tuvo que quedar en el dominio caliente. La primera idea, poner optoacopladores en el Cerebro, era mala: un optoacoplador necesita alimentar su LED desde una tension referida al chasis, lo que obligaria a meter un rail caliente dentro del Cerebro y acabaria con la idea de que el Cerebro es todo frio.

Ademas, la PCB de E/S es la que cambia con cada vehiculo, asi que interesa que añadir entradas y salidas nuevas no toque el Cerebro.

## Decision

La PCB de E/S lleva su **propio microcontrolador**, pequeño y determinista, que absorbe los interruptores como entradas digitales, el acelerador con su ADC interno y los smart switches con su bus serie local. Se comunica con el Cerebro por un unico enlace.

Candidatos: STM32C071, STM32C031 o STM32G0B1, a elegir en la Fase 1 segun disponibilidad en JLCPCB. El STM32C071 es el favorito por tener **todos sus GPIO tolerantes a 5 V**, lo que hace mucho mas robusto leer el mazo, ademas de 128 KB de flash y grado de temperatura de 125 grados.

## Alternativas consideradas

**Poner el ESP32-C6 en la PCB de E/S** y que hiciera de gestor de entradas y salidas ademas de radio. Tentador, pero se descarto por cuatro motivos: dejaria al Cerebro sin Bluetooth ni actualizacion por aire cuando no hubiera placa de E/S; pondria el mando de par detras de un SoC con pila de radio que puede colgarse; convertiria en placa de radiofrecuencia la que mas veces se va a rediseñar y que ademas conmuta 3 A de faro; y agravaria el problema de la antena dentro de la carcasa metalica. Ver ADR-0009.

**Registro de desplazamiento mas ADC serie mas aislador de seis canales**, sin microcontrolador. Cuesta practicamente lo mismo y da mucha menos flexibilidad.

## Consecuencias

- Añadir entradas y salidas para un vehiculo nuevo toca **solo la PCB de E/S y su firmware**.
- La placa puede **describirse a si misma** al arrancar, lo que resuelve gratis el problema de identificacion sin ninguna memoria (ver ADR-0010).
- El Cerebro no sabe ni necesita saber que tipo de smart switch hay detras de cada canal logico.
- Hay un **tercer firmware que mantener y actualizar**. Se resuelve en cascada: la app actualiza al ESP32, el ESP32 al STM32 y el STM32 al microcontrolador de la E/S.
- El mando de par atraviesa una cadena mas larga, lo que obliga al perro guardian, al tiempo de guarda y a las comprobaciones de plausibilidad descritas en el protocolo.
