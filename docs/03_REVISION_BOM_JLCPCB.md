# Revision de BOM y huellas para JLCPCB

Fecha de revision: 2026-07-13.

## Alcance

Se revisaron las hojas activas `ESP32.kicad_sch` y `SENSORES.kicad_sch`.
La hoja `01_ALIMENTACION.kicad_sch` fue retirada porque ya no pertenece a la
jerarquia del proyecto. `J5` esta marcado como no montar y no entra en la BOM
de produccion. Los headers THT se compran y sueldan manualmente: aparecen en
la PCB para que JLCPCB fabrique sus agujeros, pero no se incluyen en la BOM ni
en la CPL de ensamblaje.

Los precios y stocks son una foto de la fecha indicada. Se deben comprobar de
nuevo justo antes del pedido.

## Componentes seleccionados

| Referencias | Parte / LCSC | Huella | Tipo | Stock / precio unitario | Nota |
|---|---|---|---|---|---|
| C1,C3,C5,C8 | CL21A106KPFNNNE / C17024 | 0805 | Extended | 1,868,129 / $0.0086 | 10 uF; C8 fue corregido desde 0603 y C1590. |
| C2,C4,C6,C9,C11,C12,C13,C17-C21 | CL10B104KA8NNNC / C1590 | 0603 | Extended | 2,069,355 / $0.0031 | 100 nF; C17-C21 comparten huella y se consolidaran al cerrar BOM. |
| C7,C10,C14 | CC0805KKX5R6BB475 / C107125 | 0805 | Extended | 8,055 / $0.0327 | 4.7 uF, X5R. |
| R10,R11,R12 | 0603WAF1001T5E / C21190 | 0603 | Basic | 15,873,089 / $0.0009 | 1 kOhm. |
| R4,R5 | 0603WAF5101T5E / C23186 | 0603 | Basic | 7,571,904 / $0.0009 | 5.1 kOhm para CC USB-C. |
| R8,R9,R13,R14,R22,R23 | 0603WAF1002T5E / C25804 | 0603 | Basic | 37,165,617 / $0.0008 | 10 kOhm. |
| R6 | 0603WAF2212T5E / C25961 | 0603 | Extended | 218,139 / $0.0010 | 22.1 kOhm del divisor CP2102N. |
| R7 | 0603WAF4752T5E / C23061 | 0603 | Extended | 62,941 / $0.0010 | 47.5 kOhm del divisor CP2102N. |
| D2 | SP0503BAHTG / C3040626 | SOT-143 | Extended | 28,729 / $0.0837 | ESD USB; colocar junto a J3. |
| D3,D4 | SS14 / C2480 | SMA | Basic | 676,057 / $0.0142 | OR entre 5V_IN y USB_VBUS. |
| Q3,Q4 | SS8050 / C2150 | SOT-23 | Basic | 3,219,017 / $0.0115 | Programacion automatica. |
| FB1 | GZ1608D601TF / C1002 | 0603 | Basic | 952,132 / $0.0063 | 600 Ohm a 100 MHz para BMI088. |
| U1 | TLV75733PDBVR / C485517 | SOT-23-5 | Extended | 43,462 / $0.1663 | LDO de 3.3 V. |
| U2 | ESP32-WROOM-32E-N8 / C701342 | ESP32-WROOM-32E | Extended | 15,745 / $3.77 | Respetar zona libre de cobre bajo la antena. |
| U3 | CP2102N-A02-GQFN24R / C969151 | QFN-24-EP 4x4 mm | Extended | 5,744 / $1.6457 | EP central a GND con vias termicas. |
| U4 | BMI088 / C194919 | Bosch LGA-16 4.5x3 mm | Extended | 2,378 / $2.6243 | Validar pin 1 y desacoples al hacer placement. |
| U5 | SN74HC595DR / C10092 | SOIC-16 | Pendiente de reconfirmar | Referencia JLC registrada | Registro de desplazamiento para el expansor de salidas. |
| U6 | ULN2003BDR / C157485 | SOIC-16 | Pendiente de reconfirmar | Referencia JLC registrada | Driver Darlington para cargas externas. |
| U7 | LM358DR / C5423 | SOIC-8 | Pendiente de reconfirmar | Referencia JLC registrada | Etapa analogica del acelerador. |
| J3 | GT-USB-7010ASV / C2988369 | USB_C_Receptacle_G-Switch_GT-USB-7010ASV | Extended | 32,723 / $0.0749 | USB-C USB 2.0. |
| SW1,SW2 | TS1201-TZ25HAM / C36936655 | Footprints:Boton_Simple | Extended | Pendiente de reconfirmar | Huella local heredada y resuelta por fp-lib-table. |
| J5 | No montar | Sin huella | Excluido | N/A | Reserva de entrada de alimentacion. |

## Hardware THT de montaje manual

| Referencias | Huella | Ensamblaje |
|---|---|---|
| J1, J9, J10 | Header vertical 1x06, P2.54 mm | Manual; excluir de BOM/CPL JLCPCB. |
| J2, J12, J13, J14 | Header vertical 1x04, P2.54 mm | Manual; excluir de BOM/CPL JLCPCB. |
| J4, J15 | Header vertical 1x03, P2.54 mm | Manual; excluir de BOM/CPL JLCPCB. |
| J7, J8, J11 | Header vertical 1x02, P2.54 mm | Manual; excluir de BOM/CPL JLCPCB. |
| J6 | Header vertical 2x04, P2.54 mm | Manual; excluir de BOM/CPL JLCPCB. |

## Estado de Economic PCBA

- Basic/Extended: se debe reconfirmar al generar la BOM final, porque se
  incorporaron U5, U6, U7 y pasivos nuevos despues de la fotografia original.
- Los headers THT no generan colocacion Economic PCBA porque se excluyen del
  ensamblaje y se sueldan manualmente.
- La estimacion exacta de cargos depende de la clasificacion vigente de JLCPCB
  al subir BOM y CPL. Cada tipo Extended puede requerir cargo de feeder.
- `J1` es el unico bloqueo de stock detectado. No se sustituyo por una huella
  vertical distinta sin su dibujo mecanico, ya que una huella incorrecta es un
  riesgo de fabricacion mayor que dejar el item pendiente.

## Verificacion mecanica pendiente para placement

1. Colocar `U2` en un borde y mantener la zona de antena sin cobre ni pistas.
2. Colocar `D2` antes de las pistas USB y adyacente a `J3`.
3. Colocar `U4` lejos de vibraciones, conectores y fuentes de calor; confirmar
   orientacion de pin 1 contra el datasheet Bosch antes de fabricar.
4. Revisar el pad expuesto de `U3` y definir vias de masa segun el datasheet.
5. Mantener los headers THT alejados de componentes altos y dejar acceso al
   lado de soldadura para el montaje manual.
