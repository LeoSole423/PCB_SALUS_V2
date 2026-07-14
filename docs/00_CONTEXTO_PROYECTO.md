# Contexto de PCB SALUS V2

## Objetivo

PCB SALUS V2 sera una nueva placa de control para el robot Ackermann SALUS.
Se inspirara en el proyecto anterior `Hoja-Base`, pero se redisenara desde los
requisitos actuales. La meta no es duplicar la V1: es conservar las ideas utiles
y eliminar los problemas detectados antes de fabricar una nueva revision.

## Arquitectura objetivo

La placa debera separar claramente responsabilidades:

```text
Jetson Orin Nano <--> enlace definido <--> ESP32 <--> sensores y actuadores
```

- La Jetson Orin Nano se encargara de ROS 2, navegacion, percepcion y logica de
  alto nivel.
- El ESP32 se encargara de I/O, adquisicion de sensores, control de tiempo real
  y comunicacion con la Jetson.
- La potencia de traccion debera permanecer separada de la logica de control.

## Requisitos ya definidos

1. Incluir una IMU en la nueva placa.
2. Incorporar una conexion USB-C con la Jetson Orin Nano.
3. Incorporar un conector para un futuro modulo GPS RTK de doble antena.
4. Usar la V1 como referencia para entender interfaces, no como esquema para
   copiar directamente.
5. Alimentar la logica desde un modulo step-down externo de 5 V. El riel de
   servos de 7 V seguira siendo externo a esta revision hasta definir sus
   conectores y su distribucion.

## Controlador principal seleccionado

La V2 usara como controlador principal el modulo `ESP32-WROOM-32E-N8` de
Espressif, con numero de parte JLCPCB `C701342`.

Motivos de la seleccion:

- Mantiene la familia ESP32 clasica de doble nucleo usada por el prototipo que
  ya controla el robot.
- Ofrece rendimiento equivalente al sistema actual para tareas FreeRTOS, PID,
  sensores y actuadores.
- Incorpora 8 MB de flash, util para firmware, OTA y registros.
- JLCPCB lo clasifica como apto para Economic PCBA, a diferencia del
  ESP32-S3-WROOM-1-N16 evaluado anteriormente.

Condiciones que se deben respetar:

- El stock y precio de `C701342` se comprobaran nuevamente al preparar la BOM y
  antes de enviar el pedido; cambian con frecuencia.
- El ESP32 clasico no tiene USB nativo. La conexion USB-C con la Jetson debera
  incluir un puente USB-a-UART u otra interfaz definida durante el diseno.
- La elegibilidad del modulo no garantiza por si sola Economic PCBA: el resto
  de los componentes y el montaje deberan cumplir las condiciones del servicio.

## Criterio de ensamblaje JLCPCB

La V2 se disenara para Economic PCBA, con componentes SMD en una sola cara de
montaje. Para controlar el costo se seguira este orden de preferencia:

1. Componentes Basic.
2. Componentes Preferred Extended, que no agregan cargo de feeder en Economic
   PCBA.
3. Componentes Extended solo cuando no exista una alternativa Basic o Preferred
   Extended que cumpla correctamente la funcion tecnica.

Las piezas Extended se aceptaran para funciones criticas, como proteccion de
sobretension, cuando sean necesarias para seguridad o confiabilidad. Cada una
debera quedar registrada con su numero JLCPCB, stock al momento de cotizar y
justificacion tecnica, porque agrega un cargo de feeder por tipo de componente.

## Decisiones pendientes

Estas decisiones se resolveran durante el diseno, antes de agregar componentes
al esquema:

| Tema | Pregunta a resolver |
|---|---|
| USB-C con Jetson | Definir si sera datos USB, alimentacion, ambos, o una interfaz de servicio. |
| IMU | Seleccionar sensor, bus, rango, ubicacion mecanica y estrategia de calibracion. |
| GPS RTK | Seleccionar modulo de doble antena, tension, protocolo, conectores y entradas de antena. |
| Alimentacion | Definir bateria, rieles requeridos, corrientes, convertidores, fusibles y protecciones. |
| Enlace Jetson-ESP32 | Definir protocolo, velocidad, recuperacion ante reinicio y estados seguros. |
| Actuadores | Definir que salidas siguen siendo necesarias y que driver/proteccion requiere cada una. |
| Pinout y conectores | Definir niveles logicos, corriente, orientacion, mating connector y cableado. |

## Arquitectura de alimentacion confirmada

La V2 recibira dos rieles ya regulados por modulos step-down externos. La PCB
no generara 7 V elevando los 5 V: cada riel llegara desde su propio convertidor,
con la bateria como fuente aguas arriba de ambos modulos.

Datos de partida conocidos:

- La bateria principal es de plomo, con 60 V nominales.
- Un step-down externo genera 19 V para la Jetson Orin Nano.
- Un step-down externo, alimentado desde 19 V, generara 5 V para la PCB.
- Un segundo camino de conversion generara 24 V desde la bateria y, desde esos
  24 V, un step-down generara 7 V para los servos.
- El riel de servos debe tolerar picos de al menos 5 A.
- La tension maxima de falla que puede llegar a la entrada de 5 V de la V2 es
  24 V.

```text
Bateria de 60 V
  |-- step-down a 19 V --> Jetson Orin Nano
  |                         `-- step-down a 5 V --> 5V_IN / USB_VBUS --> OR D3/D4 --> 5V_SYS
  |                                                                                     |-- modulos de 5 V
  |                                                                                     `-- regulador 3.3 V --> ESP32 y logica
  |
  `-- step-down a 24 V --> step-down a 7 V --> entrada 7 V --> +7V_SERVO
                                                                  `-- conectores de servos
```

El enfoque conserva la separacion funcional de `Hoja-Base`, que tenia una
entrada protegida de 5 V para sistema y otra protegida de 7 V para servos.
La V2 se redisenara desde esa idea; no se copiaran sin validar los MOSFET,
diodos zener y valores de resistencia de la V1.

En particular, la V1 usa MOSFET `IRFR5305TRPBF`, con una clasificacion maxima
de `VDS = -55 V`. Ese valor no es apto para usar como barrera frente a una
bateria de 60 V nominales, porque no deja margen para la tension de carga
completa ni para transitorios. La V2 podria recuperar una funcion de corte por
sobrevoltaje en el futuro, pero no existe ese bloque en el esquema activo.

### Alcance de cada riel

- `+5V_SYS`: alimentara modulos que requieran 5 V. No sera generado por la
  ESP32 ni debera circular a traves de un pin de la ESP32.
- `+3V3_LOGIC`: se derivara de `+5V_SYS` con un regulador dedicado y alimentara
  la ESP32, IMU y logica compatible con 3.3 V.
- `+7V_SERVO`: alimentara solo los conectores de servo y sus cargas asociadas.
  No se usara como fuente de la logica.
- La potencia de traccion y los drivers de motor permanecen fuera de estos
  rieles, salvo que se defina expresamente un bloque dedicado en el futuro.

### Estado actual de la entrada de 5 V

La hoja de alimentacion heredada fue retirada. En el esquema actual `J5` es una
reserva sin huella, MPN ni LCSC PN; no se montara ni se incluira al enviar la
BOM a JLCPCB. Todavia se debe decidir si la entrada final sera un conector de
cable, un borne, un JST u otra interfaz.

El esquema mantiene el OR de alimentacion `D3/D4`: permite alimentar la logica
desde `5V_IN` o desde `USB_VBUS` sin retroalimentar la otra fuente. No es una
proteccion frente a sobretension, polaridad inversa ni cortocircuito.

Por una decision de esta iteracion, la V2 no incorpora aun fusible, TVS de
entrada ni corte de sobretension. Si el step-down fallara y entregara 24 V, la
placa no queda protegida por este esquema. Esa proteccion debe residir en un
step-down robusto o volver a agregarse como un bloque dedicado antes de conectar
la placa al robot.

### Limite de responsabilidad para el riel de 19 V

La V2 no distribuira ni protegera los 19 V de la Jetson. Esta placa es el
controlador de ESP32, sensores y senales; recibira solamente sus rieles de 5 V
y 7 V ya regulados. La proteccion de 19 V para la Jetson, si se implementa,
pertenecera a un modulo o placa de distribucion de potencia independiente.

La seleccion del conector de entrada, la proteccion ante 24 V y el presupuesto
de corriente siguen pendientes. `5V_IN` identifica la fuente externa y
`5V_SYS` es el riel comun despues del OR; ningun otro bloque debe asumir que
`5V_IN` es un riel protegido.

### Datos pendientes de potencia

- Medir o confirmar la tension maxima de bateria con carga completa. Los 60 V
  nominales no bastan para seleccionar los convertidores que estan conectados
  directamente a la bateria.
- Identificar modelo y numero de servos, para estimar corriente continua y
  corriente de bloqueo, y confirmar si el objetivo de 5 A de pico debe cambiar.

## Lecciones de la V1

La revision anterior mostro varios puntos que la V2 debe evitar:

- No avanzar a fabricacion con errores DRC; en particular, las separaciones de
  cobre y zonas deben quedar resueltas antes de generar Gerbers.
- Dimensionar la alimentacion y la termica con corrientes reales. Un LDO no debe
  usarse sin comprobar su disipacion y margen termico.
- Proteger interfaces externas, especialmente USB-C, contra ESD y ruido.
- Incluir fiduciales, puntos de prueba y reglas de fabricacion desde el inicio
  si se piensa usar montaje automatizado.
- Guardar MPN, fabricante, datasheet, huella y notas de ensamblaje para cada
  componente critico desde que se agrega al esquema.
- Separar retornos y rutas de potencia de motor/servos de las senales sensibles,
  del ESP32, de la IMU y de la Jetson.

## Principio de seguridad de potencia

La V2 no manejara potencia de traccion directamente, salvo que un requisito
posterior lo defina de forma explicita. Si se agrega esa funcion, se disenarara
como un bloque dedicado con driver apropiado, limites de corriente, protecciones,
retorno de masa controlado y validacion termica.

## Forma de trabajo

El diseno avanzara por bloques pequenos: requisitos, esquema, revisiones ERC,
asignacion de huellas, PCB, DRC y fabricacion. Cada decision relevante se
registrara en esta carpeta antes de quedar reflejada en el esquema.
