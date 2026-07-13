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
5. Alimentar la placa desde dos modulos step-down externos: uno de 5 V para el
   sistema y uno de 7 V para los servos.

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
  |                         `-- step-down a 5 V --> entrada 5 V de V2 --> proteccion 5 V --> +5V_SYS
  |                                                                                             |-- modulos de 5 V
  |                                                                                             `-- regulador 3.3 V --> ESP32 y logica
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
completa ni para transitorios. La V2 conservara la funcion de corte por
sobrevoltaje, pero usara una topologia y componentes clasificados para el peor
caso real.

### Alcance de cada riel

- `+5V_SYS`: alimentara modulos que requieran 5 V. No sera generado por la
  ESP32 ni debera circular a traves de un pin de la ESP32.
- `+3V3_LOGIC`: se derivara de `+5V_SYS` con un regulador dedicado y alimentara
  la ESP32, IMU y logica compatible con 3.3 V.
- `+7V_SERVO`: alimentara solo los conectores de servo y sus cargas asociadas.
  No se usara como fuente de la logica.
- La potencia de traccion y los drivers de motor permanecen fuera de estos
  rieles, salvo que se defina expresamente un bloque dedicado en el futuro.

### Proteccion prevista en la entrada de 5 V

La entrada de 5 V tendra un bloque de proteccion antes de conectarse a
`+5V_SYS`. En esta iteracion del esquema se contempla:

1. Conector generico de dos pines, con polaridad identificada. Su MPN, huella
   y conector complementario se definiran con el cableado final.
2. Proteccion contra polaridad inversa, preferentemente con MOSFET de baja
   perdida o un controlador de diodo ideal.
3. Corte por sobretension inspirado en la topologia de V1.

No se agregara fusible o PTC en esta etapa, por decision de diseno. Esto deja
sin resolver la proteccion ante un cortocircuito interno: se dimensionara y
seleccionara al cerrar el presupuesto de corriente de la PCB, GPS RTK, IMU y
demas modulos. Tambien quedan pendientes el TVS, los capacitores de entrada y
los puntos de prueba del riel.

Ademas de la polaridad inversa, este bloque debera desconectar la electronica
si la salida del step-down de 5 V falla y aparece una tension superior a la
permitida. En el robot ya ocurrio un fallo de este tipo, donde llegaron 24 V a
la entrada prevista para 5 V y se dano la electronica. La proteccion de la V1
con dos MOSFET P-channel, zener y resistencias se toma como referencia de esta
funcion de corte por sobretension.

No alcanza con un MOSFET de polaridad inversa, un zener de compuerta o un TVS:
por si solos no constituyen un corte de sobretension regulado. La V2 debera
usar una topologia equivalente a la V1 o un controlador dedicado de sobretension,
con componentes clasificados para la mayor tension de fallo que pueda llegar a
la entrada de la PCB.

### Limite de responsabilidad para el riel de 19 V

La V2 no distribuira ni protegera los 19 V de la Jetson. Esta placa es el
controlador de ESP32, sensores y senales; recibira solamente sus rieles de 5 V
y 7 V ya regulados. La proteccion de 19 V para la Jetson, si se implementa,
pertenecera a un modulo o placa de distribucion de potencia independiente.

La proteccion de 5 V sigue siendo necesaria: protege la V2 si el step-down de
5 V falla y deja pasar su tension de entrada hacia la placa.

El bloque de 5 V se dimensionara para soportar 24 V continuos en su entrada de
falla. Sus semiconductores, capacitores y resistencias criticas tendran una
clasificacion de al menos 40 V para conservar margen de diseno.

Para los dos MOSFET P-channel del corte por sobretension de 5 V se selecciona
el mismo MPN usado en la V1: `IRFR5305TRPBF` de Infineon, JLCPCB/LCSC `C2624`.
Es un DPAK (TO-252AA) de 55 V, adecuado para la falla maxima definida de 24 V.
Es Extended, pero ambos MOSFET usan el mismo modelo y por tanto comparten un
solo cargo de feeder en Economic PCBA. Se verificara simbolo, pinout y huella
contra el datasheet antes de pasarlo a PCB.

La entrada de 7 V se conectara directamente a `+7V_SERVO`, sin fusible, TVS ni
proteccion contra polaridad inversa en la PCB, por decision de diseno. Se
mantendran capacitores de reserva cerca de los conectores de servo. El conector,
cobre y vias de este riel se dimensionaran para los picos de al menos 5 A.

La entrada no protegida se denominara `VIN_5V` y se usara exclusivamente dentro
de la hoja de alimentacion cuando se actualice el conector desde KiCad.
`+5V_SYS` sera el unico riel de 5 V disponible en las demas hojas del proyecto.
El conector, los componentes de proteccion de 5 V y la corriente nominal se
seleccionaran cuando se conozcan los consumos de modulos y el cableado definitivo.

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
