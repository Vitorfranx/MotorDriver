# ADR-0003: Drivers de puerta aislados como estandar de plataforma

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

Habia dos formas de cruzar la barrera con las seis señales PWM: usar drivers de puerta aislados, donde la barrera esta dentro del propio driver, o usar un unico aislador digital hexacanal seguido de drivers de medio puente clasicos con bootstrap en el lado caliente, que es bastante mas barato.

La plataforma debe escalar hasta 500 A y 144 V.

## Decision

Se adoptan **drivers de puerta aislados como estandar de la plataforma**, desde el primer prototipo. La escalada de 48 V a 144 V sera un cambio de **alimentacion**, no de topologia: en el patinete, driver aislado con bootstrap en el lado alto; en la placa de 500 A, el mismo concepto pero con fuentes aisladas independientes por interruptor y polarizacion bipolar.

Requisitos obligatorios del driver: **salidas en estado bajo por defecto ante entrada flotante**, bloqueo por subtension, y desemparejamiento de retardo entre canales especificado.

## Alternativas consideradas

**Aislador hexacanal mas drivers bootstrap.** Mas barato y perfectamente valido a 48 V y 20 A, pero se cae por su propio peso al escalar, por cinco motivos independientes:

1. **Ciclo de trabajo.** El condensador de bootstrap solo se recarga cuando conduce el interruptor de abajo. Con indice de modulacion alto, una fase puede quedarse cerca del 100 %, el condensador se vacia y el driver entra en bloqueo por subtension justo en el peor momento.
2. **Potencia de puerta.** Con seis a diez MOSFET en paralelo por interruptor, la carga de puerta ronda los 2 o 3 microculombios, lo que a 16 kHz son cerca de 0,7 W por interruptor con picos de varios amperios. Un diodo y un condensador no dan eso de forma fiable.
3. **Polarizacion negativa de apagado.** Con muchos FET en paralelo hace falta apagar con tension negativa para evitar conduccion cruzada por efecto Miller, y el bootstrap estructuralmente no puede darla porque su referencia es el propio nodo de conmutacion.
4. **Rebote negativo del nodo de conmutacion.** Los drivers de bootstrap tipo HVIC, con su desplazador de nivel referido a masa, se enganchan o mueren cuando el nodo de fase se hunde por debajo de masa, cosa que con 500 A y cualquier inductancia parasita ocurre en cada tiempo muerto.
5. **Protecciones.** Deteccion de desaturacion, apagado suave, clamp activo de Miller y linea de fallo que ya cruza la barrera vienen de serie en un driver aislado.

## Consecuencias

- El esquema mental y el contrato con el Cerebro no cambian entre placas de potencia.
- Cada PCB de Potencia debe **declarar el retardo de propagacion de sus drivers**, porque de el depende la calibracion del tiempo muerto.
- La eleccion concreta de driver sigue siendo interna de cada PCB de Potencia: el Cerebro solo saca seis señales de nivel logico, una de deshabilitacion y recibe una de fallo.
