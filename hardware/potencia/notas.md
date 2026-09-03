# PCB Potencia

**Dominio**: atraviesa la barrera de aislamiento. Es la unica placa que lo hace.
**Varia entre vehiculos**: si.
**Papel adicional**: base mecanica del conjunto, atornillada al disipador.

---

# Revision 1 — Prototipo de patinete (48 V)

## 1. Especificacion electrica

| Parametro | Valor |
|---|---|
| Tension nominal de bateria | 48 V |
| Tension maxima de bus (la que corresponde a 2,0 V de la señal normalizada) | 60 V |
| Corriente de fase continua | 10 A eficaces |
| Corriente de fase de pico | 20 A eficaces |
| Corriente instantanea de pico de diseño | 28 A |
| Tension de los interruptores | 80 a 100 V |

## 2. Etapa de potencia

- MOSFET de 80 a 100 V. Modelo por decidir en la Fase 1.
- Tres **drivers de puerta aislados de doble canal** (ver ADR-0003), con bootstrap en el lado alto, que a 48 V y 20 A es suficiente.
  - Candidato principal: **NSI6602** de NOVOSENSE, con 4 A de fuente y 6 A de sumidero, 25 ns de retardo, 5 ns maximos de desemparejamiento entre canales y 150 kV/us de inmunidad a transitorios de modo comun. Hay stock confirmado en LCSC.
  - **Requisitos obligatorios**: salidas en estado bajo por defecto ante entrada flotante, bloqueo por subtension y desemparejamiento de retardo especificado.
- Condensadores de bus: mezcla de electrolitico y film, con el rizado calculado en la Fase 3.
- Precarga del bus, fusible y proteccion contra inversion de polaridad.

## 3. Medida de corriente

Tres sensores de fase con salida **normalizada y ratiometrica**: 0 A en VREF/2, fondo de escala en 30 A (ver ADR-0004).

- Candidato principal: **ACS725 de mas o menos 30 A**, 3,3 V, 120 kHz de ancho de banda, AEC-Q100. En SOIC-16 da 1097 V eficaces de aislamiento basico, mas que suficiente.
- **Plan B si no hay disponibilidad**: shunt mas amplificador aislado (NSI1200 o similar, con entrada de mas o menos 250 mV y ganancia 8).

**Nota para el firmware**: el ACS725 tiene 2 microsegundos de retardo de propagacion y 4 de tiempo de respuesta. A 16 kHz eso es alrededor del 6 % del periodo y desplaza el instante de muestreo respecto al centro del PWM. Hay que compensarlo.

Los sensores se alimentan del **mismo LDO de precision que genera VREF+** en el Cerebro.

## 4. Medida de tension de bus

Divisor resistivo en el lado caliente mas **amplificador aislado** (ver ADR-0005). Candidatos: NSI1311 de NOVOSENSE o AMC1311 de Texas Instruments.

Divisor dimensionado para que **2,0 V de salida correspondan a 60 V de bus**.

## 5. Medida de temperatura

Dos NTC sobre el disipador, **electricamente aisladas** del circuito de potencia mediante almohadilla termica aislante o encapsulado aislado, cableadas directamente al lado frio (ver ADR-0006).

Se mide el disipador, no la union. El modelo termico del firmware debe tenerlo en cuenta.

## 6. Alimentaciones

Dos railes necesarios:

- **12 V calientes**, referidos al negativo de bateria, para los drivers de puerta y para las cargas de la PCB de E/S (hasta 5 A en el patinete).
- **Secundario aislado** para alimentar la PCB Cerebro.

**Decision abierta**: flyback con dos secundarios (mas barato, un solo convertidor, pero regulacion cruzada mediocre) frente a buck no aislado mas un modulo aislado pequeño independiente (mas simple de diseñar, mas caro en modulos).

Ademas hace falta un LDO minusculo en el lado caliente para alimentar el amplificador aislado de la tension de bus.

## 7. Hardware de barrera alojado en esta placa

Ver ADR-0002. Toda la barrera vive aqui:

- Los tres drivers de puerta aislados.
- Los tres sensores de corriente (que son su propia barrera).
- El amplificador aislado de la tension de bus.
- El **aislador digital de 6 canales** del enlace con la PCB de E/S: TX, RX, NRST, BOOT0, apagado de salidas, y un canal de reserva.
- La fuente aislada.

## 8. Parametros que esta placa debe declarar

Obligatorio para toda PCB de Potencia, presente o futura (ver ADR-0002 apartado 5.3 de la arquitectura):

| Parametro | Valor en la revision 1 |
|---|---|
| Corriente de fondo de escala | 30 A |
| Tension maxima de bus | 60 V |
| Retardo de propagacion de los drivers | Por medir en la Fase 3 |
| Coeficientes de las NTC | Por definir |
| Codigo de identificacion cableado | Por asignar en la Fase 2 |

---

# Notas para la futura placa de alta potencia (144 V, 500 A)

Recogidas aqui para que no se pierdan.

- **La tecnologia del sensor de corriente cambia por completo.** La familia ACS725 se acaba en mas o menos 40 A. La respuesta para ese rango es la familia **coreless** de Allegro (ACS37612 y ACS37610), sensores Hall diferenciales que no se conectan electricamente a nada, sino que se montan sobre un busbar o sobre una pista de cobre y miden el campo. Cubren de 100 A a mas de 4000 A con precision tipica del 1 %, en TSSOP-8, con 240 a 250 kHz de ancho de banda. El aislamiento es inherente y las perdidas nulas. El ACS37610 ademas tiene un **pin de fallo** que dispara por sobrecorriente y sobretemperatura, ideal para cortar el puente por hardware. Alternativa clasica: serie HST de LEM, hasta 1500 A.
- Gracias a ADR-0004, ese cambio de sensor **no afecta al Cerebro ni al firmware**.
- **Los drivers necesitaran fuentes aisladas independientes por interruptor y polarizacion bipolar** (por ejemplo mas 15 y menos 8 V), y conviene subir a un driver con deteccion de desaturacion.
- Habra seis a diez MOSFET en paralelo por interruptor, con carga de puerta total del orden de 2 a 3 microculombios.
- El cobre grueso limita las opciones de montaje en JLCPCB. Verificar antes de diseñar.
