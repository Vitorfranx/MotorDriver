# Protocolo del enlace Cerebro - PCB E/S

> **Estado**: especificacion funcional cerrada. El formato binario exacto de cada mensaje se detalla en la Fase 4, al escribir el firmware.

## 1. Capa fisica

| Parametro | Valor |
|---|---|
| Tipo | UART asincrono, full duplex |
| Velocidad | 1 Mbaudio |
| Formato | 8 bits de datos, sin paridad, 1 bit de parada |
| Aislamiento | Aislador digital de 6 canales, en la PCB de Potencia |
| Reloj | El microcontrolador de la PCB E/S lleva **cristal externo obligatorio** |

### Por que cristal obligatorio

Los candidatos de la familia STM32C0 **no tienen PLL** y su oscilador interno es de mas/menos 1 %. Dos osciladores a mas/menos 1 % pueden separarse un 2 %, que esta dentro de lo que tolera un UART pero sin margen comodo. El cristal cuesta veinte centimos y elimina la duda.

### Por que UART y no SPI

Se evaluaron ambos. El UART gano por tres motivos:

1. **Se resincroniza solo.** Un esclavo SPI que pierda la alineacion de bytes por un pico de ruido puede quedarse desincronizado indefinidamente. Un UART recupera la sincronia en cada bit de arranque, y el STM32 ofrece ademas deteccion de linea en reposo y bandera de error de trama.
2. **Gasta dos canales de aislador en lugar de cuatro**, lo que deja sitio dentro de un aislador de seis para NRST, BOOT0 y la linea de apagado de salidas.
3. **No obliga a doble buffer en el esclavo.** Un esclavo SPI tiene que tener la respuesta cargada en DMA antes de que empiece la trama, lo que introduce un retardo inherente de una trama y es una fuente clasica de errores sutiles.

El determinismo que se pierde frente al SPI es teorico: a 1 kHz de sondeo, ambos dan la misma latencia.

## 2. Capa de enlace

- **El Cerebro es siempre quien inicia.** La PCB de E/S nunca transmite sin haber sido preguntada.
- Ciclo de **1 kHz**.
- Trama de longitud fija con: cabecera de sincronismo, identificador de mensaje, contador de secuencia, carga util y **CRC-16**.
- El contador de secuencia permite detectar tramas perdidas, repetidas o desordenadas.

## 3. Mensajes

| Mensaje | Sentido | Cuando |
|---|---|---|
| Descubrimiento | Cerebro pregunta | Al arrancar |
| Autodescripcion | E/S responde | Respuesta al descubrimiento |
| Estado ciclico | E/S responde | Cada milisegundo |
| Mando de salidas | Cerebro envia | Cada milisegundo |
| Entrada en cargador de arranque | Cerebro envia | Solo durante una actualizacion |

### 3.1 Autodescripcion

La PCB de E/S declara:

- Tipo de placa, revision de hardware y numero de serie.
- Version de su firmware.
- Tabla de salidas: para cada canal, nombre logico, tipo y corriente maxima.
- Tabla de entradas: para cada canal, nombre logico y tipo.

Esto sustituye a la memoria de identificacion que se descarto: **la placa se describe a si misma**. El Cerebro lo reenvia a la app movil, que construye su interfaz dinamicamente segun el vehiculo conectado.

El Cerebro no sabe ni necesita saber si detras de un canal de salida hay un smart switch con interfaz serie o un conmutador discreto. Esa eleccion es interna de cada PCB de E/S.

### 3.2 Estado ciclico

- Estado de los cinco interruptores, ya interpretados segun su configuracion de normalmente abierto, normalmente cerrado o deshabilitado.
- Valor del acelerador y su estado de diagnostico.
- Valor medido del propio rail de 5 V del acelerador.
- Diagnostico de cada salida: normal, circuito abierto, cortocircuito a masa, cortocircuito a bateria, sobrecarga, sobretemperatura.

### 3.3 Mando de salidas

Estado deseado de cada canal de salida, referido a **canales logicos**, no a pines fisicos.

## 4. Comportamiento ante fallo del enlace

Es simetrico y es la red de seguridad principal del sistema.

| Quien detecta | Condicion | Reaccion |
|---|---|---|
| PCB E/S | 5 tramas consecutivas sin recibir mando | Apaga **todas** las salidas y deja de importar el acelerador |
| Cerebro | 5 tramas consecutivas sin respuesta | Fuerza **par cero** |

Con esto, un fallo del microcontrolador de la E/S, de su reloj, de su alimentacion o del propio cable acaba siempre en estado seguro.

Ademas existe la linea fisica de **apagado de salidas**, independiente del firmware de ambos lados, como ultimo recurso.

## 5. Consideracion de seguridad sobre el acelerador

El acelerador vive en la PCB de E/S, asi que el mando de par atraviesa una cadena: acelerador, ADC del microcontrolador de E/S, UART, aislador, STM32. Eso obliga a:

- Perro guardian en el microcontrolador de la E/S.
- Comprobacion de banda valida del acelerador: se considera valido entre 0,5 y 4,5 V; por debajo de 0,3 V se interpreta como cable cortado o cortocircuito a masa, y por encima de 4,7 V como cortocircuito a 5 V.
- Exigencia de que el acelerador lea reposo al encender antes de permitir traccion.
- Prioridad absoluta del freno sobre el acelerador.
- El tiempo de guarda descrito en el apartado 4.

## 6. Actualizacion del firmware de la PCB E/S

No hace falta escribir un cargador de arranque propio. El STM32 del Cerebro gobierna las lineas **NRST y BOOT0** del microcontrolador de la E/S a traves del aislador, lo que le permite meterlo en el **cargador de arranque de fabrica** y reprogramarlo por el mismo UART. La ventaja decisiva es que funciona **aunque el firmware de la E/S este corrupto**.
