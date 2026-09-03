# ADR-0008: Enlace Cerebro-E/S por UART aislado

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

Con un microcontrolador propio en la PCB de E/S (ADR-0007), habia que elegir como se comunican las dos placas a traves de la barrera. Se evaluaron UART, SPI y CAN.

## Decision

**UART asincrono a 1 Mbaudio**, con trama ciclica de longitud fija a 1 kHz iniciada siempre por el Cerebro, con contador de secuencia y CRC-16. El microcontrolador de la E/S lleva **cristal externo obligatorio**.

## Alternativas consideradas

**SPI con el Cerebro de maestro.** Su ventaja es que el reloj lo pone el maestro y no hay que preocuparse de la precision del oscilador, cosa relevante porque el STM32C0 no tiene PLL y su oscilador interno es de mas o menos 1 %. Se descarto por tres motivos:

1. **No se resincroniza.** Un esclavo SPI que pierda la alineacion de bytes por un pico de ruido puede quedarse desincronizado indefinidamente. Un UART recupera la sincronia en cada bit de arranque, y ademas el STM32 ofrece deteccion de linea en reposo y bandera de error de trama. En un enlace que cruza una barrera dentro de un inversor conmutando, esa capacidad de recuperarse sola tiene valor real.
2. **Gasta cuatro canales de aislador en lugar de dos.** Con UART caben, dentro de un aislador de seis canales, las lineas de NRST, BOOT0 y apagado de salidas. Con SPI harian falta siete canales.
3. **Obliga a doble buffer en el esclavo**, que tiene que tener la respuesta cargada en DMA antes de que empiece la trama. Eso introduce un retardo inherente de una trama y es una fuente clasica de errores sutiles.

Ademas, el retardo de ida y vuelta a traves del aislador limitaria la frecuencia de reloj SPI a unos pocos MHz.

**CAN.** Habria permitido colgar cajas de entradas y salidas remotas, lejos del driver, con solo dos hilos. Descartado por decision expresa del usuario: el driver debe ser autocontenido, sin modulos adicionales.

## Consecuencias

- El cristal en la PCB de E/S es obligatorio, no opcional.
- El determinismo que se pierde frente al SPI es teorico: a 1 kHz de sondeo, ambos dan la misma latencia.
- Queda libre un SPI en el microcontrolador de la E/S por si una placa futura lo necesita.
- Las lineas NRST y BOOT0 permiten reprogramar el microcontrolador de la E/S con su **cargador de arranque de fabrica**, sin escribir ni una linea de bootloader propio, y rescatarlo aunque su firmware este corrupto.
- El tiempo de guarda es simetrico y es la red de seguridad principal: cinco tramas perdidas hacen que la E/S apague todas las salidas y que el Cerebro fuerce par cero.
