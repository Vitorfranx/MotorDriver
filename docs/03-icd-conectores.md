# Documento de interfaz de conectores (ICD)

> **Estado**: el contrato electrico esta cerrado. El pinout concreto se fija en la **Fase 2** y hasta entonces las tablas de este documento son esqueletos.

El proposito de este documento es que el conector entre placas sea un **contrato versionado y congelado**: una PCB de Potencia de 144 V diseñada dentro de dos años debe seguir encajando con el Cerebro de hoy.

**Version del ICD**: 0.1 (borrador, sin pinout)

---

## 1. Interfaz Cerebro - Potencia

### 1.1 Naturaleza

Solo lleva **logica y alimentacion de bajo consumo**, incluso en la futura placa de 500 A. Toda la corriente de potencia se queda en la PCB de Potencia y no pasa nunca por este conector.

### 1.2 Señales previstas

| Grupo | Cantidad | Sentido | Dominio | Notas |
|---|---|---|---|---|
| PWM | 6 | Cerebro a Potencia | Frio | Nivel logico 3,3 V |
| Deshabilitacion / STO | 1 | Cerebro a Potencia | Frio | Activo a nivel bajo, seguro ante desconexion |
| Fallo de driver | 1 | Potencia a Cerebro | Frio | Salida del lado frio de los drivers |
| Corrientes de fase | 3 | Potencia a Cerebro | Frio | Analogicas normalizadas, 0 A en VREF/2 |
| Tension de bus | 1 | Potencia a Cerebro | Frio | Analogica, 0 a 2 V |
| Temperaturas NTC | 2 | Potencia a Cerebro | Frio | Divisor en el Cerebro |
| Identificacion de placa | 4 | Potencia a Cerebro | Frio | Codigo cableado, a masa o al aire |
| UART hacia la E/S | 5 | Ambos | Frio | Pasa a traves de la Potencia hacia el aislador |
| Alimentacion aislada | 2 o mas | Potencia a Cerebro | Frio | Del secundario aislado |
| Masas | Varias | — | Frio | Intercaladas entre analogicas y PWM |

Total estimado: **de 34 a 40 posiciones**.

### 1.3 Reglas de trazado obligatorias

- Intercalar pines de masa entre las señales analogicas de corriente y las señales PWM, para evitar diafonia. Las corrientes de fase son señales analogicas en un entorno con dv/dt muy alto y son lo mas delicado de todo el conector.
- El pin de deshabilitacion debe llevar el sistema a estado seguro si el conector se desconecta.

### 1.4 Pinout

*(Pendiente. Se fija en la Fase 2, junto con la asignacion de pines del STM32.)*

| Pin | Señal | Sentido | Notas |
|---|---|---|---|
| | | | |

---

## 2. Interfaz Potencia - E/S

### 2.1 Naturaleza

Es el unico conector entre placas que lleva corriente apreciable: el rail de 12 V caliente que alimenta las cargas del vehiculo.

### 2.2 Señales previstas

| Grupo | Cantidad | Notas |
|---|---|---|
| 12 V calientes | Varios pines | Hasta 5 A en el patinete (3 + 1 + 1). Contar unos 3 A por pin de paso 2,54 mm, o usar un conector de potencia aparte |
| Masa caliente | Varios pines | Mismo dimensionado que el positivo |
| UART aislado (lado caliente) | 2 | TX y RX ya cruzados |
| NRST del MCU de E/S | 1 | Gobernado por el Cerebro |
| BOOT0 del MCU de E/S | 1 | Gobernado por el Cerebro |
| Apagado de salidas | 1 | Linea fisica, independiente del firmware |

### 2.3 Decision de diseño

El **aislador digital se coloca en la PCB de Potencia**, no en la de E/S. De ese modo la PCB de E/S es 100 % caliente y trivialmente barata, lo cual importa porque es la placa de la que se van a hacer mas variantes. El coste es que toda placa de Potencia lleva el aislador aunque no se use placa de E/S.

### 2.4 Pinout

*(Pendiente. Se fija en la Fase 2.)*

| Pin | Señal | Notas |
|---|---|---|
| | | |

---

## 3. Conectores externos del Cerebro

Todos previstos como bornes **push-in**.

| Conector | Posiciones | Señales |
|---|---|---|
| Hall | 5 | H1, H2, H3, +5 V, masa |
| Encoder | 8 | 3 pares diferenciales, +5 V, masa |
| CAN | 3 | CAN_H, CAN_L, malla |

Total aproximado: **16 bornes**.

### 3.1 El conector de encoder es unico para tres interfaces

Los ocho pines cubren los tres modos, seleccionados por firmware mediante las habilitaciones de los transceptores RS-485:

| Modo | Par 1 | Par 2 | Par 3 |
|---|---|---|---|
| Incremental TTL diferencial | A, A negada (recepcion) | B, B negada (recepcion) | Z, Z negada (recepcion) |
| BiSS-C | Reloj MA (emision) | Datos SLO (recepcion) | Sin usar |
| SSI | Reloj (emision) | Datos (recepcion) | Sin usar |

Se conservan los **ocho pines aunque hoy sobre uno**, para que una futura placa adaptadora de encoder senoidal de 1 Vpp pueda enchufarse en el mismo borne sin rediseñar el Cerebro.

Terminacion fija de 120 ohmios por par, junto al conector.

## 4. Propuestas de apilado mecanico

*(Pendiente de decision. Recogidas aqui para la Fase 2.)*

**Propuesta preferida — dos satelites sobre una base**: la PCB de Potencia es la placa grande, atornillada al disipador, y tanto el Cerebro como la E/S se montan sobre ella, uno al lado del otro, cada uno con su conector. Permite colocar el Cerebro sobre la zona fria, lejos del puente, y quitar la E/S sin tocar el control. En una placa de 500 A habra sitio de sobra.

**Alternativa — sandwich completo**: Potencia abajo, Cerebro encima, E/S encima del Cerebro. Huella minima, pero el Cerebro recibe el calor de los interruptores y desmontar la E/S obliga a desmontar todo.

**Descartada — coplanar unida por cable o flex**: introduce cables donde hay señales analogicas sensibles y dv/dt alto.

Sobre el tipo de conector: un conector de dos filas a paso 2,54 mm con separadores es lo mas tolerante a vibracion y a errores de alineamiento para un prototipo. Un mezzanine de paso fino es mas compacto pero exige tolerancias mecanicas finas.
