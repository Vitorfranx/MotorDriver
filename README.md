# Driver de motor modular

Driver de motor trifasico con control FOC, construido como **plataforma modular** de tres PCBs, pensado para adaptarse a distintos vehiculos cambiando solo dos de ellas.

El primer prototipo es para un **patinete electrico de 48 V**, con 10 A eficaces continuos y 20 A eficaces de pico por fase. La arquitectura esta pensada para escalar hasta **144 V y 500 A** por fase reutilizando la misma PCB de control.

## Antes de trabajar en este repositorio

Lee siempre, por este orden:

1. [`docs/00-ESTADO.md`](docs/00-ESTADO.md) — que esta hecho, que queda pendiente y que decisiones siguen abiertas.
2. [`docs/02-arquitectura.md`](docs/02-arquitectura.md) — la arquitectura de dominios y la barrera de aislamiento.
3. [`docs/decisiones/`](docs/decisiones/) — el porque de cada decision cerrada.

Al terminar cada sesion de trabajo, **actualiza `docs/00-ESTADO.md`**.

## Las tres PCBs

| PCB | Dominio | Papel | Varia entre vehiculos |
|---|---|---|---|
| **Cerebro** | Frio (flotante) | STM32G474QET6, FOC, sensores de motor, Bluetooth, CAN | No |
| **Potencia** | Atraviesa la barrera | Puente de interruptores, sensores de corriente, alimentaciones. Base mecanica del conjunto | Si |
| **E/S** | Caliente (negativo de bateria) | Interruptores del manillar, acelerador, luces y bocina | Si |

## Estructura del repositorio

```
docs/            Documentacion viva: requisitos, arquitectura, interfaces, decisiones
hardware/        Listas de materiales y notas de diseno de cada PCB
firmware/        Proyectos de firmware (Cerebro, E/S, ESP32-C6)
app/             Aplicacion movil
Documentacion/   Datasheets y manuales de referencia
```

## Herramientas

- **Esquematicos y PCB**: los disena el usuario por su cuenta.
- **Fabricacion y montaje**: JLCPCB. Toda la lista de materiales debe estar restringida a su biblioteca de componentes.
- **Configuracion del MCU**: STM32CubeMX.
- **Firmware**: STM32CubeIDE.
