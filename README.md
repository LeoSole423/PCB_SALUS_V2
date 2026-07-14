# PCB SALUS V2

Nueva version de la PCB de control para el robot SALUS.

El proyecto se crea desde requisitos actuales y toma como referencia funcional la
placa `Hoja-Base` anterior. No se copiara el diseno anterior como bloque cerrado:
cada subsistema se redisenara, validara y documentara antes de pasar a PCB.

## Abrir el proyecto

En KiCad, abre `PCB_SALUS_v2.kicad_pro`.

El proyecto usa KiCad 10 y ya contiene las hojas `ESP32` y `SENSORES`, las
librerias locales y la documentacion de las decisiones tomadas. Todavia esta en
desarrollo: no debe enviarse a fabricar hasta resolver ERC, conectorizacion,
placement, ruteo y DRC.

## Documentacion

- [Puente H BTS7960](docs/04_PUENTE_H_BTS7960.md)

- [Contexto del proyecto](docs/00_CONTEXTO_PROYECTO.md): objetivo, requisitos
  conocidos, decisiones pendientes y lecciones de la V1.
- [Estructura del esquematico y librerias](docs/01_ESTRUCTURA_ESQUEMATICO_Y_LIBRERIAS.md): como separar la V2 en hojas y gestionar componentes locales.
- [Validacion y pruebas](docs/02_VALIDACION_Y_PRUEBAS.md): resultados de
  simulacion, pruebas pendientes y plan de validacion fisica.

## Estado actual

La arquitectura y el controlador principal ya estan definidos: ESP32-WROOM-32E-
N8 de Espressif (`C701342` en JLCPCB), por compatibilidad con el prototipo y
Economic PCBA. La IMU BMI088, USB-C y el puente USB-UART estan en el esquema.

El GPS RTK, los conectores de actuadores restantes y la proteccion final de
alimentacion continuan pendientes de especificacion.
