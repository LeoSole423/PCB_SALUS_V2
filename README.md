# PCB SALUS V2

Nueva version de la PCB de control para el robot SALUS.

El proyecto se crea desde requisitos actuales y toma como referencia funcional la
placa `Hoja-Base` anterior. No se copiara el diseno anterior como bloque cerrado:
cada subsistema se redisenara, validara y documentara antes de pasar a PCB.

## Abrir el proyecto

En KiCad, abre `PCB_SALUS_v2.kicad_pro`.

El proyecto comienza vacio en KiCad 10. Todavia no contiene componentes,
esquema, huellas ni librerias locales. Esto es intencional: primero se definiran
los requisitos y la arquitectura.

## Documentacion

- [Contexto del proyecto](docs/00_CONTEXTO_PROYECTO.md): objetivo, requisitos
  conocidos, decisiones pendientes y lecciones de la V1.
- [Estructura del esquematico y librerias](docs/01_ESTRUCTURA_ESQUEMATICO_Y_LIBRERIAS.md): como separar la V2 en hojas y gestionar componentes locales.

## Estado actual

La primera etapa es documentar y acordar la arquitectura. El controlador
principal seleccionado es el ESP32-WROOM-32E-N8 de Espressif (`C701342` en
JLCPCB), por compatibilidad con el prototipo y Economic PCBA.

Todavia no se debe seleccionar un IMU, modulo GPS RTK, conector USB-C ni
componente de potencia sin definir antes su funcion, voltajes, protocolo y
requisitos de integracion.
