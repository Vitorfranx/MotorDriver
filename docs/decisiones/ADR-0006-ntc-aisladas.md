# ADR-0006: NTC aisladas fisicamente y leidas directamente

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

Las temperaturas de los interruptores se miden en el lado caliente, pero el ADC esta en el frio. Igual que con la tension de bus, el usuario prefirio lectura directa sin bus serie.

## Decision

Montar las NTC **electricamente aisladas del circuito de potencia** (con almohadilla termica aislante contra el disipador, o con NTC de encapsulado aislado) y cablearlas directamente al lado frio. **No se usa ningun componente de aislamiento.**

## Alternativas consideradas

- **NTC pegada al encapsulado del MOSFET**, que esta al potencial del nodo de conmutacion. Da la medida mas cercana a la union, pero obliga a aislar electronicamente cada canal.
- **Oscilador controlado por la NTC cruzando un canal de aislador digital**, midiendo la frecuencia con un temporizador. Cuesta menos de un euro y es muy inmune al ruido. Queda anotada como opcion si algun dia hiciera falta medir en el propio encapsulado.
- **Sensor digital aislado sobre bus serie.** Descartado por la misma preferencia del usuario.

## Consecuencias

- Coste cero en componentes de aislamiento.
- Se mide el **disipador y no la union**, que es lo que se hace habitualmente en gestion termica. El modelo termico del firmware debe tenerlo en cuenta.
- Hay que respetar distancias de fuga y de aislamiento en el montaje de la NTC.
- En la futura placa de 500 A, si se usa un modulo de potencia, lo mas probable es que ya traiga una NTC integrada y aislada de los chips, con lo que esta decision sale gratis.
