# ADR-0014: FRAM para parametros y registro de vida

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

El STM32G474 **no tiene EEPROM**. La configuracion que llega desde la app (ver ADR-0010) tiene que guardarse en algun sitio no volatil del Cerebro, y ademas interesa registrar contadores de vida y un historial de fallos.

## Decision

Montar una **FRAM en el Cerebro**, conectada por I2C. Referencia de partida: **FM24CL64B-G** de Infineon (64 kbit organizados como 8K x 8, SOIC-8, I2C hasta 1 MHz, 2,7 a 3,65 V), con el pin de proteccion de escritura gobernado por un GPIO para que el firmware solo desbloquee la escritura cuando esta guardando de forma intencionada.

Alternativas equivalentes: MB85RC256V de Fujitsu (256 kbit, I2C) o FM25V02 (256 kbit, SPI).

## Alternativas consideradas

- **EEPROM I2C clasica** (familia 24xx). Cuesta unos 30 centimos frente a 1,50 o 2,30 euros de la FRAM, pero soporta del orden de un millon de escrituras por celda y necesita 5 ms por pagina.
- **Emulacion en flash del propio STM32.** No añade componentes, pero da unos 10.000 ciclos por pagina y obliga a gestionar borrados. El doble banco del G4 permite leer del otro banco mientras se borra, pero sigue siendo engorroso.

## Consecuencias

- **Resistencia practicamente ilimitada** (del orden de 10^13 ciclos): se puede escribir el cuentakilometros, la energia consumida, los picos de temperatura o un registro rodante de los ultimos segundos antes de un fallo cada 10 ms durante toda la vida del producto.
- **Escribe a velocidad de bus, sin retardo.** Esto es lo decisivo en un driver: cuando se detecta una caida de alimentacion, da tiempo a volcar el estado con la energia que queda en el condensador de la fuente. Con una EEPROM, esos 5 ms puede que no esten.
- Conexion trivial: alimentacion, masa, SDA y SCL con dos resistencias de subida, tres pines de direccion y el de proteccion de escritura. Ni cristal ni componentes externos.
- Cuesta unas cinco veces mas que una EEPROM, lo que en la lista de materiales de este driver es ruido.
