# ADR-0009: ESP32-C6 en el Cerebro para Bluetooth y actualizacion por aire

**Fecha**: 3 de septiembre de 2026
**Estado**: Aceptada

## Contexto

El STM32G474 no tiene radio, asi que hace falta un modulo externo. Los requisitos son: app en movil (lo que obliga a Bluetooth de baja energia, porque iOS no permite el perfil serie clasico sin programa MFi), configuracion, diagnostico, telemetria y actualizacion del firmware por aire.

## Decision

**Modulo ESP32-C6 precertificado, montado en la PCB Cerebro**, con **conector U.FL y antena externa**.

El modulo descarga la imagen completa del firmware en su propia flash y **verifica su integridad antes** de tocar el STM32.

## Alternativas consideradas

- **STM32WB5MMG de ST.** Coherencia total con CubeMX y CubeIDE, modulo certificado con antena, y ejemplos de actualizacion por aire del propio fabricante. Se descarto por precio y por tener una comunidad mucho menor.
- **Modulo nRF52840.** La mejor pila BLE del mercado y el mayor caudal, lo que haria la actualizacion mas rapida. Se descarto por añadir una tercera cadena de herramientas.
- **Modulo BLE de perfil serie transparente**, tipo RN4870. Minimo esfuerzo de desarrollo, pero caudal pobre (una actualizacion de 256 KB tardaria varios minutos), poco control sobre la estructura GATT y seguridad dependiente del fabricante.
- **Colocarlo en la PCB de E/S**, ver ADR-0007.

## Consecuencias

- Es el mas barato del grupo y le sobra flash (4 MB) para almacenar la imagen entera del STM32.
- Regala WiFi, lo que abre una segunda via de actualizacion y diagnostico, e incluso una interfaz web de configuracion sin escribir app movil.
- Hay un **segundo proyecto de firmware** con otra cadena de herramientas (ESP-IDF), y el propio modulo necesita poder autoactualizarse.
- **La antena externa no es opcional.** Si el driver va en una carcasa de aluminio que hace de disipador, una antena de pista no radia. Es de las cosas que arruinan un prototipo si se descubren tarde.
- El modulo necesita alimentacion de 3,3 V capaz de dar picos de varios cientos de miliamperios.
- **Riesgo de seguridad a cubrir**: activar o desactivar el driver desde el movil es un vector de ataque. Hace falta emparejamiento seguro, y la salvaguarda de que no se pueda detener la traccion con el vehiculo en marcha.
