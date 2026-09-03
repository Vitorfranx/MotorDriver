# ADR-0012: Descartar el encoder de 1 Vpp en la revision 1

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

El requisito inicial incluia soportar encoders senoidales de 1 Vpp. Es, con diferencia, el interfaz mas caro y complejo de los pedidos.

Un encoder de 1 Vpp entrega señales diferenciales de 1 V de pico a pico centradas en unos 2,5 V, y obtener el angulo exige un front-end analogico completo mas interpolacion por arcotangente. La calidad de la medida no depende de la calidad absoluta de cada canal, sino de las **diferencias entre A y B**: si tienen ganancias distintas, la figura de Lissajous se convierte en una elipse y el angulo sale con error periodico; si tienen offsets distintos, el circulo se descentra y ocurre lo mismo.

## Decision

**Se descarta el 1 Vpp en la revision 1** de la PCB Cerebro. Se conservan, sin embargo, **los ocho pines del conector de encoder**, para que una futura placa adaptadora pueda enchufarse en el mismo borne sin rediseñar el Cerebro.

## Alternativas consideradas

- **Tres amplificadores de instrumentacion.** Es el bloque funcionalmente ideal, porque su ganancia la fija una sola resistencia con las internas ajustadas de fabrica, lo que da un emparejamiento entre canales muy superior al de un diferencial hecho con resistencias discretas. Pero exige vigilar tres cosas: ancho de banda y sobre todo **fase** (si A y B se desfasan distinto con la frecuencia, el error angular crece con la velocidad); **rango de modo comun**, porque con 2,5 V de modo comun y 3,3 V de alimentacion un instrumental clasico de tres operacionales se queda sin margen y habria que alimentarlo a 5 V; y que la marca de referencia no necesita precision y podria ir a un comparador interno del G4.
- **Los seis operacionales internos del STM32G474** con resistencias externas. Salen gratis, pero con peor CMRR y emparejamiento.
- **Modo diferencial del ADC del G4** con red pasiva. Elimina los amplificadores, pero 1 Vpp sobre un fondo de escala de mas o menos 3,3 V usa el 15 % del rango y pierde casi tres bits.

Cualquiera de ellas seguiria necesitando calibracion por software de ganancia, offset y cuadratura para dar buenos resultados.

## Consecuencias

- **Simplifica mucho la placa mas cara del conjunto**: desaparecen tres amplificadores de instrumentacion, los conmutadores analogicos que hacian falta para que el camino analogico y el digital convivieran sobre los mismos pines, tres canales de ADC y el uso de los operacionales internos.
- El front-end del encoder se queda en tres transceptores y sus terminaciones.
- **La puerta sigue abierta** gracias a los ocho pines conservados.
