# ADR-0015: Restriccion de la lista de materiales a la biblioteca de JLCPCB

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

Las PCBs se van a fabricar **y montar** en JLCPCB. Muchos de los componentes que aparecen de forma natural al diseñar un driver de motor aislado son piezas de grado automocion de Infineon, Allegro o Texas Instruments, y ese es justo el catalogo que JLCPCB peor cubre.

Ademas, el inventario de LCSC y el de JLCPCB son **independientes y no se traspasan**: ver una pieza en LCSC no significa que se pueda montar.

## Decision

- La **referencia de JLCPCB es una columna obligatoria** de toda lista de materiales, desde el primer componente.
- **Ningun componente se da por cerrado** sin haber verificado que existe en la biblioteca de JLCPCB, y sin haber anotado una alternativa al lado.
- La verificacion de disponibilidad se hace en la **Fase 1, antes de dibujar ningun esquematico**.

## Alternativas consideradas

**Diseñar con los componentes tecnicamente ideales y resolver la disponibilidad al final.** Es lo natural y es exactamente el error que esta decision pretende evitar: el filtro cuesta poco mientras diseñas y mucho cuando ya has dibujado.

## Consecuencias

- Puede obligar a elegir componentes tecnicamente inferiores. Es un precio asumido.
- Hallazgo util derivado de esta restriccion: el fabricante **NOVOSENSE** cubre practicamente toda la cadena de aislamiento con piezas bien surtidas en LCSC y a precios muy bajos. El NSI6602 es un driver de puerta aislado de doble canal con 4 A de fuente y 6 A de sumidero, 25 ns de retardo, 5 ns maximos de desemparejamiento entre canales, 150 kV/us de inmunidad a transitorios de modo comun, control de tiempo muerto y **salidas en estado bajo por defecto ante entrada flotante**, por menos de la mitad de precio que un UCC21520. El NSI1311 es sustituto directo del AMC1311. El mismo fabricante tiene aisladores digitales, transceptores CAN aislados y amplificadores aislados para shunt.
- Hay que tener presente que los componentes **"extendidos" llevan tasa de preparacion por referencia distinta**, y que el montaje por las dos caras encarece bastante, lo que puede empujar a concentrar componentes en una sola cara.
- Cuando una pieza no este en la biblioteca, quedan los servicios de **pedido anticipado** y de **abastecimiento global** de JLCPCB, que la compran y la guardan en la biblioteca privada del usuario.
