# PCB Cerebro

**Dominio**: frio (flotante).
**Varia entre vehiculos**: no. Es la placa invariante de la plataforma.

> Cualquier propuesta que obligue a modificar esta placa para adaptarse a un vehiculo concreto va en contra de la arquitectura y debe señalarse como tal.

## 1. Microcontrolador

**STM32G474QET6**, LQFP128 de 14 x 14 mm.

| Recurso | Valor |
|---|---|
| Nucleo | Cortex-M4F a 170 MHz |
| Flash | 512 KB, doble banco read-while-write |
| RAM | 128 KB (96 + 32 de CCM) |
| Entradas y salidas | 107 |
| ADC | 5 convertidores de 12 bits, 42 canales, con sobremuestreo por hardware |
| Comparadores | 7 |
| Operacionales | 6 |
| Temporizadores | 17, incluidos 3 avanzados de control de motor y el HRTIM |
| FDCAN | 3 |
| SPI | 4 |
| USART | 5 mas 1 LPUART |
| Aceleradores | CORDIC y FMAC |

**Atencion al grado de temperatura**: el sufijo `6` es de menos 40 a 85 grados. Existen los grados de 105 y 125 grados, a considerar dado que la placa va dentro de una caja con interruptores calientes.

**Disponibilidad**: figura en la biblioteca de JLCPCB como `C730129`, pero LCSC lo lista como pieza de pedido anticipado. Verificar en la Fase 1.

## 2. Referencia del ADC

LDO de precision de 3,3 V que alimenta **VREF+** y tambien los sensores de corriente y el amplificador aislado de la PCB de Potencia, para que la cancelacion ratiometrica funcione (ver ADR-0004).

**Restriccion critica**: el STM32 exige **VREF+ menor o igual que VDDA en todo momento, incluido el arranque**. Si VDDA y VREF+ vienen de dos LDO distintos, ambos a 3,3 V nominales, las tolerancias pueden hacer que VREF+ supere a VDDA momentaneamente. Hay que decidir en la Fase 3 como se secuencian o se relacionan esos railes.

## 3. Bluetooth

Modulo **ESP32-C6 precertificado con conector U.FL y antena externa** (ver ADR-0009). La antena externa no es opcional: dentro de una carcasa de aluminio, una antena de pista no radia.

Enlace por USART con el STM32, mas control de reset y de arranque para poder actualizarlo.

Necesita alimentacion de 3,3 V capaz de dar picos de varios cientos de miliamperios.

## 4. Memoria no volatil

**FRAM I2C** (FM24CL64B-G o equivalente), con el pin de proteccion de escritura gobernado por GPIO (ver ADR-0014).

## 5. Sensores de posicion

### 5.1 Hall

Tres entradas asimetricas, tipicamente de colector abierto con resistencia de subida a 5 V. Requieren adaptacion a 3,3 V y deben ir a pines **tolerantes a 5 V** que a la vez esten conectados a canales del temporizador que se use en modo Hall.

Borne push-in de 5 posiciones: H1, H2, H3, 5 V y masa.

### 5.2 Encoder

**Tres transceptores RS-485** de alta velocidad con **true failsafe**, uno por par diferencial, con habilitaciones independientes por firmware (ver ADR-0011).

Un unico borne push-in de **8 posiciones** que cubre incremental TTL diferencial, BiSS-C y SSI. Terminacion fija de 120 ohmios por par, junto al conector.

Se conservan los ocho pines aunque hoy sobre uno, para permitir una futura placa adaptadora de 1 Vpp (ver ADR-0012).

Requisito de velocidad: el BiSS-C puede pedir hasta 10 MHz de reloj. Compromiso a decidir en la Fase 3: los transceptores rapidos tienen flancos rapidos y por tanto mas emisiones radiadas.

### 5.3 Alimentacion del encoder

Rail de **5 V del lado frio**, protegido contra cortocircuito y **con medida de su consumo**: un cable pelado en el motor no debe llevarse la placa por delante, y saber cuanta corriente pide el encoder es un diagnostico gratuito.

## 6. Comunicaciones

- **CAN aislado** hacia el BMS: transceptor aislado, borne push-in de 3 posiciones (CAN_H, CAN_L, malla).
- **Enlace con la PCB de E/S**: un USART, que sale por el conector hacia la Potencia donde esta el aislador (ver ADR-0002 y ADR-0008).

## 7. Lo que esta placa NO lleva

- **Ninguna entrada ni salida del mazo del vehiculo.** Ni acelerador, ni interruptores, ni luces. Todo eso vive en la PCB de E/S, en el dominio caliente. Es precisamente lo que hace invariante a esta placa.
- Ningun front-end de 1 Vpp en la revision 1.

## 8. Otros

- Conector SWD para programacion y depuracion.
- LEDs de estado.
- 4 entradas para el codigo de identificacion cableado de la PCB de Potencia (ver ADR-0010).

## 9. Estimacion de ocupacion

Entre 60 y 70 pines de los 107 disponibles. Cabe con holgura.

Bornes push-in en el borde de la placa: unos 16 (5 de Hall, 8 de encoder, 3 de CAN).
