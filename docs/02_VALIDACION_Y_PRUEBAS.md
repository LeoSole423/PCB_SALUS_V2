# Validacion y Pruebas

## Proposito

Este documento es el registro vivo de lo que se reviso, simulo y probo en la
PCB SALUS V2. Su objetivo es evitar confundir una idea de diseno, una
simulacion y una prueba fisica en el robot.

Estados usados:

- `Pendiente`: aun no se reviso ni se probo.
- `Simulado`: se evaluo con un modelo SPICE; no reemplaza una prueba fisica.
- `Revisado`: se verifico esquema, valores o huellas, sin prueba electrica.
- `Probado en banco`: se midio sobre una placa real alimentada de forma segura.
- `Probado en robot`: se verifico en las condiciones de uso del robot.

## Entorno de simulacion

- Herramienta: KiCad 10.0.4.
- Motor SPICE: ngspice 42.
- Fecha de la primera corrida: 2026-07-13.
- Alcance actual: pasivos, redes RC, divisores y modelos genericos de diodos o
  transistores. No hay modelos SPICE especificos guardados aun en el proyecto.

## Resultados actuales

| Bloque | Estado | Resultado | Limite de confianza |
|---|---|---|---|
| RC de RESET, R22 + C8 | Simulado | Frecuencia de corte: 1.588 Hz para 10 kOhm y 10 uF. | Modelo ideal de R y C; falta prueba de programacion real. |
| Divisor USB VBUS, R6 + R7 | Simulado | Con 5.0 V de entrada, salida calculada: 3.412 V. | Modelo ideal y sin carga del pin VBUS del CP2102N. |
| Desacople de +3.3V | Simulado | Impedancia estimada: 0.009 Ohm a 1 MHz. | Estimacion con ESR/ESL genericos; no incluye PCB ni componentes reales. |
| Desacople de CP2102_3V3 | Simulado | Impedancia estimada: 0.036 Ohm a 1 MHz. | Estimacion generica. |
| Desacople de 5V_SYS | Simulado | Impedancia estimada: 0.019 Ohm a 1 MHz. | Estimacion generica. |
| Desacople de USB_VBUS | Simulado | Impedancia estimada: 0.036 Ohm a 1 MHz. | Estimacion generica. |
| Arranque ideal de +3.3V | Simulado | El modelo simplificado llega a 3.3 V; corriente de irrupcion estimada: 0.133 A. | No representa el lazo de control real del TLV757. |
| Transistores Q3/Q4 de auto-programacion | Simulado | El modelo NPN generico conmuta. | Falta evaluar la tabla DTR/RTS completa y el MPN real. |
| OR de 5 V, D3/D4 | Simulado | A 100 mA, una fuente de 5 V entrega 4.693 V en 5V_SYS; a 500 mA entrega 4.628 V. Con una fuente a 5.2 V y la otra a 5.0 V, 5V_SYS queda en 4.893 V. | Modelo Schottky generico; falta validar Vf y temperatura del SS14 seleccionado. |
| Aislamiento entre USB y 5V_IN | Simulado | Con 5V_IN a 5 V, USB_VBUS a 0 V y carga de 100 mA, la corriente hacia USB fue aproximadamente 2 uA de fuga generica, no una retroalimentacion de potencia. | Falta medicion con los diodos reales. |
| Tabla DC de auto-programacion | Simulado | DTR/RTS altos o bajos a la vez: EN y BOOT altos. DTR alto/RTS bajo: EN = 31 mV y BOOT alto. DTR bajo/RTS alto: EN alto y BOOT = 31 mV. | No valida aun la secuencia temporal completa de esptool ni el CP2102N real. |
| Margen del TLV75733 tras el OR | Revisado | El peor caso simulado de 4.628 V deja 0.903 V por encima de 3.3 V. El dropout maximo publicado a 1 A es 0.425 V, por lo que el LDO sigue en regulacion en esta condicion simulada. | Requiere medir la tension real del step-down o USB, la caida real del SS14 y el consumo maximo. |

## Pendientes de simulacion

| Prioridad | Bloque | Objetivo | Requisito previo |
|---|---|---|---|
| Alta | OR de alimentacion D3/D4 | Medir en banco la caida de tension y fuga inversa con el SS14 real, especialmente con carga alta. | Placa fisica y fuente limitada en corriente. |
| Alta | Auto-programacion Q3/Q4 | Simular la secuencia transitoria usada por esptool y probar la carga real de firmware por USB. | Definir niveles y tiempos del CP2102N; luego placa fisica. |
| Alta | Proteccion de entrada de 5 V | Decidir si la proteccion frente a una falla de 24 V se implementa en el step-down o vuelve como bloque dedicado en la PCB. | Requisito de seguridad y topologia definida. |
| Media | TLV75733 | Revisar arranque y respuesta a escalones de carga. | Modelo SPICE oficial del TLV75733 o modelo equivalente validado. |
| Media | ESD USB-C | Revisar la estrategia de proteccion de VBUS, D+ y D-. | Modelo de SP0503BAHTG; la validacion definitiva requiere prueba ESD fisica. |

## Bloques que no se pueden validar completamente por SPICE hoy

- ESP32-WROOM-32E: requiere firmware, programacion real, RF y pruebas de I/O.
- CP2102N: USB, enumeracion y programacion se validaran con un PC y una placa.
- USB-C: la conexion, CC, ESD y mecanica necesitan revision de PCB y prueba con
  cables y hosts reales.
- TLV75733: sin el modelo especifico del fabricante, la estabilidad del LDO no
  debe considerarse validada.
- Proteccion contra sobretension: un modelo generico de MOSFET no basta para
  aprobar el umbral de corte ni el comportamiento ante una falla de 24 V.

## Plan de prueba fisica

Antes de conectar una Jetson, servos o sensores, probar la placa con una fuente
de laboratorio limitada en corriente.

1. Alimentar VIN_5V con 5 V y limite inicial de corriente bajo.
2. Medir 5V_SYS y +3.3V antes de instalar o alimentar cargas externas.
3. Verificar que USB y VIN_5V no se retroalimenten entre si.
4. Conectar USB-C a un PC y confirmar que el CP2102N enumera correctamente.
5. Probar carga de firmware por USB, reset y BOOT automaticos.
6. Aumentar la carga de 3.3 V de forma controlada y observar tension y
   temperatura del TLV757.
7. Probar cada conector de sensores y actuadores por separado.
8. Repetir las pruebas integradas en el robot con ruedas elevadas y parada de
   emergencia disponible.

## Criterio de actualizacion

Al completar una prueba, agregar fecha, condiciones, instrumentos usados,
resultado medido y cualquier diferencia respecto al comportamiento esperado.
Un bloque solo pasara a `Probado en robot` despues de superar antes la prueba
en banco.
