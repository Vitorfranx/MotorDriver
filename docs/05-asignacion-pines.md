# Asignacion de pines

> **Estado**: esqueleto. Se rellena en la **Fase 2**, en paralelo con la configuracion del proyecto en STM32CubeMX.
>
> Este documento es la **fuente de verdad legible** de la asignacion de pines. El fichero `.ioc` de CubeMX es la fuente de verdad ejecutable. Ambos deben mantenerse coherentes, y cualquier cambio en uno obliga a actualizar el otro en el mismo commit.

---

## 1. STM32G474QET6 (PCB Cerebro)

Encapsulado LQFP128 de 14 x 14 mm, 107 entradas y salidas disponibles.

**Estimacion de ocupacion**: entre 60 y 70 pines de los 107, con holgura suficiente.

### 1.1 Restricciones conocidas antes de asignar

- Las tres entradas Hall deben estar en pines **tolerantes a 5 V** y a la vez conectados a canales del temporizador que se use en modo Hall.
- Las seis salidas PWM deben salir de un temporizador avanzado con generacion de tiempo muerto y entrada de parada de emergencia.
- Las tres corrientes de fase deben poder muestrearse **simultaneamente**, aprovechando que el G474 tiene cinco convertidores.
- La entrada de fallo debe poder atacar la entrada de parada de emergencia del temporizador, para cortar el PWM por hardware sin intervencion del software.
- El SPI del encoder solo usa SCK y MISO (el reloj hace de MA y la entrada recoge SLO).
- **VREF+ debe ser menor o igual que VDDA en todo momento, incluido el arranque.**
- El DMA del encoder debe tener prioridad superior al del enlace con la E/S.

### 1.2 Tabla de asignacion

| Pin | Puerto | Funcion | Periferico | Notas |
|---|---|---|---|---|
| | | | | |

### 1.3 Resumen de periféricos previstos

| Periferico | Uso |
|---|---|
| Temporizador avanzado | 6 PWM con tiempo muerto y parada de emergencia |
| Temporizador | Modo Hall |
| Temporizador | Modo encoder incremental (A y B) |
| ADC (varios) | 3 corrientes de fase simultaneas, tension de bus, 2 NTC |
| SPI | Encoder BiSS-C y SSI (solo SCK y MISO) |
| USART | Enlace con la PCB E/S |
| USART | Enlace con el modulo ESP32-C6 |
| I2C | FRAM |
| FDCAN | BMS, con transceptor aislado |
| CORDIC | Funciones trigonometricas del FOC |

---

## 2. Microcontrolador de la PCB E/S

> Pendiente de elegir el modelo concreto (Fase 1, segun disponibilidad en JLCPCB).

### 2.1 Restricciones conocidas

- Las cinco entradas de interruptor deben ir a pines **con capacidad de ADC**, para dejar abierta una futura codificacion resistiva de tres estados.
- El acelerador necesita un canal de ADC, y su propio rail de 5 V otro canal mas como diagnostico.
- NRST y BOOT0 deben estar accesibles desde el conector, gobernados por el Cerebro.
- Cristal externo obligatorio.

### 2.2 Tabla de asignacion

| Pin | Puerto | Funcion | Periferico | Notas |
|---|---|---|---|---|
| | | | | |
