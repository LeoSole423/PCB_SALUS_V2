# Mapa de submodulos y pinout

Fecha de actualizacion: 2026-07-14.

## Proposito y alcance

Este documento describe la conectividad actualmente dibujada entre las hojas
`ESP32.kicad_sch` y `SENSORES.kicad_sch`. Es una referencia para firmware,
cableado y placement. No es un informe ERC/DRC, una validacion electrica ni
una simulacion; esos controles se realizaran en una etapa posterior.

## Convencion de conectores

Todos los conectores genericos son headers THT verticales de paso 2.54 mm. JLCPCB
fabricara los pads y agujeros metalizados, pero estos headers se compran y
suelden manualmente. Deben excluirse de la BOM y CPL de ensamblaje JLCPCB.

| Referencias | Huella |
|---|---|
| J1, J9, J10 | `PinHeader_1x06_P2.54mm_Vertical` |
| J2, J12, J13, J14 | `PinHeader_1x04_P2.54mm_Vertical` |
| J4, J15 | `PinHeader_1x03_P2.54mm_Vertical` |
| J7, J8, J11 | `PinHeader_1x02_P2.54mm_Vertical` |
| J6 | `PinHeader_2x04_P2.54mm_Vertical` |

`J3` es la excepcion: es un receptaculo USB-C SMD para montaje por JLCPCB.
`J5` permanece como DNP, sin huella ni compra.

## Alimentacion y nucleo ESP32

- La PCB recibe 5 V externos y tambien puede recibir `USB_VBUS`. D3 y D4
  forman el OR por diodos que alimenta `5V_SYS`.
- U1, `TLV75733PDBVR`, genera `+3.3V` para U2 y la logica de sensores.
- U2 es `ESP32-WROOM-32E-N8`. Sus buses y señales de aplicacion se distribuyen
  mediante etiquetas globales hacia la hoja `SENSORES`.

| Funcion | Red | ESP32-WROOM-32E |
|---|---|---|
| I2C SDA | `I2C_SDA` | GPIO21 |
| I2C SCL | `I2C_SCL` | GPIO22 |
| SPI MOSI | `SPI_MOSI` | GPIO23 |
| SPI MISO | `SPI_MISO` | GPIO19 |
| SPI reloj | `SPI_CLOCK` | GPIO18 |
| BMI088 CS acelerometro | `CS_ACCEL` | GPIO4 |
| BMI088 CS giroscopio | `CS_GIRO` | GPIO26 |
| BMI088 interrupcion acelerometro | `DR_ACCEL` | GPIO34 |
| BMI088 interrupcion giroscopio | `DR_GIRO` | GPIO35 |
| PWM puente, lado L | `L_PWM` | GPIO33 |
| PWM puente, lado R | `R_PWM` | GPIO27 |
| Limite izquierdo | `FC_LEFT` | GPIO36 / SENSOR_VP |
| Limite derecho | `FC_RIGHT` | GPIO39 / SENSOR_VN |
| Hall A | `HALL_A` | GPIO5 |
| Hall B | `HALL_B` | GPIO13 |
| Hall C | `HALL_C` | GPIO15 |
| Entrada RF | `RF_PPM` | GPIO32 |
| Salida DAC | `DAC` | GPIO25 |
| Registro de reles | `RELAY_LATCH` | GPIO12 |
| Habilitacion de reles | `RELAY_OE` | GPIO14 |
| UART0 TX | `U0TX` | GPIO1 |
| UART0 RX | `U0RX` | GPIO3 |
| UART1 TX | `U1_TX` | GPIO17 |
| UART1 RX | `U1_RX` | GPIO16 |
| Arranque | `BOOT` | GPIO0 |
| Reset | `RESET` | EN |

## USB-C, USB-UART y programacion

### USB-C y CP2102N

`J3` usa el receptaculo USB-C `GT-USB-7010ASV`. Sus pares USB 2.0 se unen como
`USB_DP` y `USB_DM`; `USB_VBUS` ingresa al OR de alimentacion y a U3. D2 es la
proteccion ESD y U3 (`CP2102N-A02-GQFN24R`) actua como puente USB-UART.

U3 intercambia UART0 con el ESP32: TXD de U3 va a `U0RX`/GPIO3 y RXD de U3 va
a `U0TX`/GPIO1. Sus señales DTR/RTS se usan junto a Q3 y Q4 para las redes
`BOOT` y `RESET`.

### Header de programacion J1

| Pin | Red | Destino |
|---|---|---|
| 1 | GND | Referencia comun |
| 2 | +5V | Riel de 5 V |
| 3 | `U0TX` | GPIO1 |
| 4 | `U0RX` | GPIO3 |
| 5 | `RESET` | EN del ESP32 |
| 6 | `BOOT` | GPIO0 del ESP32 |

### UART1 J4

| Pin | Red | ESP32 |
|---|---|---|
| 1 | `U1_TX` | GPIO17 |
| 2 | `U1_RX` | GPIO16 |
| 3 | GND | GND |

## IMU e I2C

### BMI088

U4 es la IMU BMI088. Usa el riel `IMU_3.3` derivado de `+3.3V` a traves de
FB1, con los desacoples C12, C13, C14, C15 y C16 presentes en esta hoja.
Comparte el bus SPI con el expansor U5 y usa dos chip-select independientes.

| Señal BMI088 | Red | ESP32 |
|---|---|---|
| SDI | `SPI_MOSI` | GPIO23 |
| SDO1 y SDO2 | `SPI_MISO` | GPIO19 |
| SCK | `SPI_CLOCK` | GPIO18 |
| CSB1 | `CS_ACCEL` | GPIO4 |
| CSB2 | `CS_GIRO` | GPIO26 |
| INT1 | `DR_ACCEL` | GPIO34 |
| INT3 | `DR_GIRO` | GPIO35 |

### AS5600 e I2C externo

J2 es el conector del modulo AS5600. J12 y J13 exponen el mismo bus I2C para
dispositivos externos. Los tres respetan el siguiente pinout.

| Pin | Red | ESP32 |
|---|---|---|
| 1 | `+3.3V` | Alimentacion de modulo |
| 2 | GND | Referencia comun |
| 3 | `I2C_SDA` | GPIO21 |
| 4 | `I2C_SCL` | GPIO22 |

## Actuadores y entradas externas

### Puente H BTS7960, J6

J6 conecta la logica de la PCB con el modulo IBT-2/BTS7960. El detalle de uso
del modulo esta en [04_PUENTE_H_BTS7960.md](04_PUENTE_H_BTS7960.md).

| Pin | Red | Funcion documentada |
|---|---|---|
| 1 | GND | Masa de logica |
| 2 | +5V | Alimentacion de logica del modulo |
| 3 | Sin etiqueta | Reserva `R_IS` / sensado de corriente |
| 4 | Sin etiqueta | Reserva `L_IS` / sensado de corriente |
| 5 | `HBRIDGE_EN` | R_EN |
| 6 | `HBRIDGE_EN` | L_EN |
| 7 | `L_PWM` | GPIO33 |
| 8 | `R_PWM` | GPIO27 |

`HBRIDGE_EN` se genera desde la salida QG de U5 y tiene R1 como elemento
asociado. No corresponde a una GPIO directa del ESP32 en el esquema actual.

### Expansor de reles

U5 (`SN74HC595DR`) recibe `SPI_MOSI`, `SPI_CLOCK`, `RELAY_LATCH` y
`RELAY_OE` desde la ESP32. Sus salidas QA a QF alimentan las entradas I1 a I6
de U6 (`ULN2003BDR`). U6 entrega seis salidas colector abierto hacia los
conectores de carga.

| Conector | Pin 1 | Pines de salida | Pin final |
|---|---|---|---|
| J9 | +5V | 2 a 5: U6 O1 a O4 | 6: GND |
| J10 | +5V | 2 a 4: U6 O5 a O7 | 5: sin etiqueta, 6: GND |

Los nombres finales de cada carga/rele no estan definidos todavia en las
etiquetas del esquema; se conservan como redes locales hasta esa definicion.

### Finales de carrera

| Conector | Pin 1 | Pin 2 | ESP32 |
|---|---|---|---|
| J7 | `FC_LEFT` | GND | GPIO36 |
| J8 | `FC_RIGHT` | GND | GPIO39 |

R2/C17 y R3/C18 son las redes asociadas a estas entradas. R2 y R3 conservan
el valor `R` y requieren valor definitivo antes de generar la BOM de compra.

### Acelerador

J11 expone `THROTTLE_OUT` y GND. La red `THROTTLE_OUT` llega a la etapa con
U7 (`LM358DR`) y sus pasivos asociados. El esquema actual mantiene esta red
como etiqueta local de la hoja `SENSORES`; su destino final se documentara en
la revision funcional posterior.

### Sensores Hall

| Pin J14 | Red | ESP32 |
|---|---|---|
| 1 | `HALL_A` | GPIO5 |
| 2 | `HALL_B` | GPIO13 |
| 3 | `HALL_C` | GPIO15 |
| 4 | GND | GND |

### Receptor RF PPM

| Pin J15 | Red | Nota |
|---|---|---|
| 1 | GND | Referencia comun |
| 2 | +5V | Alimentacion del receptor |
| 3 | Entrada a R21/R24 | La red se acondiciona hacia `RF_PPM`, GPIO32 |

## Componentes agregados a montaje SMD

| Ref. | Parte | LCSC PN | Huella |
|---|---|---|---|
| U5 | SN74HC595DR | C10092 | SOIC-16 3.9 x 9.9 mm, paso 1.27 mm |
| U6 | ULN2003BDR | C157485 | SOIC-16 3.9 x 9.9 mm, paso 1.27 mm |
| U7 | LM358DR | C5423 | SOIC-8 3.9 x 4.9 mm, paso 1.27 mm |
| C17 a C21 | 100 nF | C1590 | 0603 |
| R1 a R3, R15 a R19, R21, R24 | Segun valor | Pendiente/consolidado despues | 0603 |

No se asigna MPN ni LCSC a R2/R3 hasta definir sus valores. Esta decision no
impide que sus pads 0603 se incorporen a la PCB.

## Siguiente etapa

Antes de fabricar se debera hacer una revision electrica independiente: ERC,
DRC, alimentacion, niveles logicos, protecciones, valores pendientes y reglas
de placement/ruteo. Este documento no reemplaza esas validaciones.
