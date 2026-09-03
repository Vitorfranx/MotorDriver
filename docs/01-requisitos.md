# Requisitos

## 1. Objetivo

Driver de motor trifasico con control vectorial (FOC), construido como plataforma modular de tres PCBs, de forma que adaptarlo a un vehiculo nuevo consista en rediseñar solo las PCBs de Potencia y de E/S, dejando la PCB Cerebro intacta.

## 2. Requisitos electricos

### 2.1 Primer prototipo (patinete electrico)

| Parametro | Valor |
|---|---|
| Tension nominal de bateria | 48 V |
| Corriente de fase continua | 10 A eficaces |
| Corriente de fase de pico | 20 A eficaces (28 A instantaneos) |
| Motor | Trifasico con sensores Hall |
| Sensor adicional para pruebas | Encoder incremental TTL diferencial |

### 2.2 Escalabilidad de la plataforma

La arquitectura debe permitir, sin rediseñar la PCB Cerebro, construir PCBs de Potencia de hasta:

| Parametro | Valor maximo previsto |
|---|---|
| Tension de bateria | 144 V |
| Corriente de fase | 500 A |

## 3. Funciones del sistema

### 3.1 Control

- Lectura de las corrientes de las tres fases.
- Generacion de PWM trifasico con tiempo muerto.
- Calculo de FOC, desarrollado desde cero (sin X-CUBE-MCSDK).
- Lectura de posicion: sensores Hall, encoder incremental TTL diferencial, BiSS-C y SSI.
- Gestion de emergencias: exceso de temperatura, sobrecorriente, sobretension, subtension, STO.

### 3.2 Entradas del vehiculo (patinete)

| Entrada | Tipo | Notas |
|---|---|---|
| Acelerador | Sensor Hall de 0 a 5 V, alimentado a 5 V | Diagnostico por banda valida |
| Freno delantero | Interruptor | Configurable NA / NC / deshabilitado |
| Freno trasero | Interruptor | Configurable NA / NC / deshabilitado |
| Luces | Interruptor | Configurable NA / NC / deshabilitado |
| Bocina | Interruptor | Configurable NA / NC / deshabilitado |
| Caballete | Interruptor | Configurable NA / NC / deshabilitado |

Todos los interruptores deben poder configurarse como normalmente abierto, normalmente cerrado o deshabilitado. Se cablean a pines con capacidad de ADC para dejar abierta, sin rediseño, una futura codificacion resistiva de tres estados que permita distinguir "abierto" de "desconectado".

### 3.3 Salidas del vehiculo (patinete)

| Salida | Corriente |
|---|---|
| Luz frontal | 3 A |
| Luz de freno | 1 A |
| Bocina | 1 A |

Todas con smart switch de diagnostico capaz de detectar circuito abierto, cortocircuito a masa, cortocircuito a bateria, sobrecarga y sobretemperatura.

### 3.4 Comunicaciones

- **Bluetooth de baja energia** con telefono movil, mediante modulo ESP32-C6 en la PCB Cerebro, para: activar y desactivar el driver, configurarlo (velocidad maxima, corriente maxima, etc.), actualizarlo por aire, diagnosticarlo y medir variables.
- La app movil, en posicion horizontal, hara de cuadro de instrumentos.
- **CAN aislado** hacia el BMS del vehiculo.

## 4. Requisitos de aislamiento

Aislamiento galvanico real entre el dominio de control y el de potencia. A 48 V no responde a una necesidad de seguridad electrica (esta por debajo de los 60 V), sino a dos motivos:

1. **Inmunidad al ruido** del inversor.
2. **Modularidad**: hace la PCB Cerebro agnostica a la tension del vehiculo, de modo que la misma placa sirva para una PCB de Potencia de 48 V y para una de 144 V.

## 5. Requisitos de seguridad funcional

- El freno tiene prioridad absoluta sobre el acelerador.
- El acelerador debe leer reposo al encender antes de permitir tracción.
- Perdida del enlace con la PCB E/S: par cero y todas las salidas apagadas.
- Toda desconexion o cortocircuito en el mazo debe ser detectable y no debe dañar ninguna placa.
- La activacion o desactivacion remota por Bluetooth no debe poder detener la traccion con el vehiculo en marcha.

## 6. Restricciones de fabricacion

Las PCBs se fabrican y montan en **JLCPCB**. En consecuencia, ningun componente puede darse por cerrado sin haber verificado que existe en su biblioteca, y toda lista de materiales incluye obligatoriamente la referencia de JLCPCB y una alternativa.

## 7. Fuera de alcance por ahora

- Encoder senoidal de 1 Vpp: descartado en la revision 1, previsto mediante una futura placa adaptadora que se conecte al mismo borne de encoder.
- Resolver.
- Intermitentes: el patinete no los lleva; se contemplaran en las PCBs de E/S de vehiculos futuros.
- Certificacion y homologacion.
