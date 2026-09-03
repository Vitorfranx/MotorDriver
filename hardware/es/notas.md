# PCB E/S

**Dominio**: caliente, referido al negativo de bateria. La placa es 100 % caliente y **no lleva ningun componente de aislamiento**.
**Varia entre vehiculos**: si. Es la placa de la que se haran mas variantes.

> Por que es caliente: los interruptores del manillar y las luces retornan por el chasis del vehiculo, que es el negativo de bateria. Se ha asumido el caso peor, que comparten masa con el chasis (pendiente de confirmar en el patinete real). En el dominio caliente esas señales no necesitan ningun aislamiento.

---

# Revision 1 — Prototipo de patinete

## 1. Microcontrolador

Lleva microcontrolador propio (ver ADR-0007). Modelo **por decidir en la Fase 1** segun disponibilidad en JLCPCB.

| Candidato | Flash / RAM | Ventajas | Inconvenientes |
|---|---|---|---|
| **STM32C071** (favorito) | 128 KB / 24 KB | **Todos los GPIO tolerantes a 5 V**, LQFP48, grado de 125 grados, 2 SPI, 2 USART mas 2 via USART | Sin PLL |
| STM32C031 | 32 KB / 12 KB | El mas barato | 32 KB se quedan justos con cargador de arranque y autodescripcion |
| STM32G0B1 | Hasta 512 KB | Mucha mas memoria y periferia | Mas caro, y sus ventajas no aportan aqui |

**Cristal externo obligatorio** (ver ADR-0008): la familia C0 no tiene PLL y su oscilador interno de mas o menos 1 % deja poco margen para un UART.

Alimentacion: desde el rail de 12 V caliente que llega de la PCB de Potencia, con su regulador a 3,3 V.

## 2. Salidas

| Canal | Carga | Corriente |
|---|---|---|
| 1 | Luz frontal | 3 A |
| 2 | Luz de freno | 1 A |
| 3 | Bocina | 1 A |

**Smart switch con diagnostico**, capaz de detectar circuito abierto (en encendido y en apagado), cortocircuito a masa, cortocircuito a bateria, sobrecarga y sobretemperatura.

- Candidato principal: **SPOC+2 de Infineon** (por ejemplo BTS72220-4ESA, 4 canales, TSDSO-24, control y diagnostico completo por bus serie, encadenable en daisy chain, logica compatible con 3,3 V). Con 4 canales cubre los 3 del patinete y deja uno de reserva.
- **Riesgo alto de disponibilidad**: es pieza de automocion y probablemente no este en JLCPCB. Plan B: PROFET discretos (un pin de control y uno de diagnostico analogico por canal) o equivalente del catalogo de JLCPCB.

**Limite importante**: los smart switch de automocion tienen tension maxima de operacion en torno a 28 V, asi que **no pueden colgar del pack de 48 V**. Van obligatoriamente al rail de 12 V.

La eleccion concreta de smart switch es **interna de esta placa y transparente para el Cerebro**, que solo conoce canales logicos.

## 3. Entradas de interruptor

| Entrada | Configuracion |
|---|---|
| Freno delantero | NA / NC / deshabilitado |
| Freno trasero | NA / NC / deshabilitado |
| Luces | NA / NC / deshabilitado |
| Bocina | NA / NC / deshabilitado |
| Caballete | NA / NC / deshabilitado |

Circuito: resistencia de subida al rail caliente, adaptacion de nivel, proteccion contra sobretension y filtro RC.

**Regla de diseño**: las cinco entradas se cablean a pines **con capacidad de ADC**, aunque hoy se usen como digitales. Hoy no cuesta nada y deja abierta, sin rediseño, una futura codificacion resistiva de tres estados que permita distinguir "interruptor abierto" de "interruptor desconectado".

Configurar el freno como normalmente cerrado es en si mismo una medida de seguridad: un cable cortado se lee como freno accionado.

## 4. Acelerador

Sensor Hall de 0 a 5 V alimentado a 5 V, leido por el **ADC interno** del microcontrolador.

Circuito: divisor (por ejemplo a 0 a 2,5 V), resistencia serie, diodos de recorte y filtro RC.

**Un segundo canal del ADC mide el propio rail de 5 V del acelerador**, lo que permite detectar que se ha hundido por un cable pelado en el manillar. Es un diagnostico dificil de conseguir de otro modo.

El rail de 5 V debe estar protegido contra cortocircuito.

Diagnostico por banda valida: se considera valido entre 0,5 y 4,5 V; por debajo de 0,3 V se interpreta como cable cortado o cortocircuito a masa; por encima de 4,7 V como cortocircuito a 5 V.

## 5. Enlace con el Cerebro

UART a 1 Mbaudio (ver ADR-0008 y el protocolo en `docs/04-protocolo-es.md`).

**NRST y BOOT0 accesibles desde el conector**, gobernados por el Cerebro, para poder reprogramar el microcontrolador con su cargador de arranque de fabrica y rescatarlo aunque su firmware este corrupto.

Linea fisica de **apagado de salidas**, independiente del firmware.

## 6. Autodescripcion

Al arrancar, la placa declara al Cerebro su tipo, revision, numero de serie, version de firmware y las tablas de sus entradas y salidas. Esto sustituye a la memoria de identificacion que se descarto (ver ADR-0010).

## 7. Comportamiento ante fallo del enlace

Cinco tramas consecutivas sin recibir mando: **apaga todas las salidas** y deja de importar el acelerador.

## 8. Lo que esta placa NO lleva

- Ningun componente de aislamiento: el aislador digital vive en la PCB de Potencia (ver ADR-0002).
- Ninguna radio. El modulo Bluetooth vive en el Cerebro (ver ADR-0009).
- Intermitentes: el patinete no los lleva. Se contemplaran en las PCBs de E/S de vehiculos futuros, que es exactamente para lo que sirve que esta placa sea barata y facil de rehacer.
