# Estructura del Esquematico y Librerias

## Objetivo

La V2 crecera por bloques. El archivo raiz mostrara la arquitectura general y
cada bloque se implementara en una hoja jerarquica independiente. Esto facilita
la lectura, las revisiones y la reutilizacion de bloques sin convertir el
esquematico en una unica hoja dificil de navegar.

## Hojas actuales

| Hoja | Responsabilidad | Estado inicial |
|---|---|---|
| `ESP32.kicad_sch` | ESP32, regulador de 3.3 V, USB-C, CP2102N, arranque y programacion. | Activa. |
| `SENSORES.kicad_sch` | BMI088, desacoples, ferrita y senales de sensores. | Activa, en desarrollo. |

Las futuras hojas deben crearse solo cuando exista un bloque con requisitos
definidos, por ejemplo GPS RTK o actuadores.

## Crear Una Hoja Jerarquica

1. Abrir el esquematico raiz `PCB_SALUS_v2.kicad_sch`.
2. Elegir `Colocar -> Anadir hoja jerarquica`.
3. Dibujar el rectangulo de la hoja en el esquema raiz.
4. En el dialogo, usar un nombre y archivo que describan el bloque, por ejemplo
   `GPS_RTK` y `GPS_RTK.kicad_sch`.
5. Hacer doble clic en la hoja para entrar y dibujar dentro el bloque nuevo.

Las conexiones entre hojas se hacen con etiquetas jerarquicas y pines de hoja.
Los rieles globales de potencia, como `GND`, `+5V_SYS` y `+3V3_LOGIC`, pueden
usar simbolos de potencia o etiquetas globales. Las senales concretas entre
bloques, como `UART_JETSON_TX`, deben cruzar mediante etiquetas jerarquicas,
para que el diagrama raiz muestre claramente cada interfaz.

## Librerias Del Proyecto

Las librerias locales pertenecen al proyecto y evitan depender de archivos
personales de otra maquina. La V2 conserva una copia local heredada en `Libs/`,
pero cada simbolo y huella que se use debe verificarse contra su MPN antes de
fabricar.

Estructura recomendada:

```text
PCB_SALUS_v2/
  Libs/
    Symb/Simbolos.kicad_sym
    Footprints/Footprints.pretty/
    3D/
```

### Crear La Libreria De Simbolos

1. Abrir `Preferencias -> Gestionar bibliotecas de simbolos`.
2. Ir a la pestana `Bibliotecas especificas del proyecto`.
3. Elegir el boton para agregar una nueva biblioteca.
4. Crear o seleccionar `Libs/Symb/Simbolos.kicad_sym`.
5. Confirmar que se agregue como biblioteca del proyecto, no global.

Usar esta biblioteca cuando un simbolo no exista en KiCad o deba representar un
componente personalizado. Para un componente estandar, conservar el simbolo de
KiCad y agregar MPN, fabricante, LCSC/JLCPCB y datasheet como propiedades del
simbolo.

### Crear La Libreria De Huellas

1. Abrir el `Editor de huellas`.
2. Elegir `Preferencias -> Gestionar bibliotecas de huellas`.
3. En `Bibliotecas especificas del proyecto`, seleccionar
   `Libs/Footprints/Footprints.pretty`.
4. Confirmar que su alias sea exactamente `Footprints`, como en `fp-lib-table`.

Guardar una huella local solo cuando la huella estandar no coincida con el
datasheet del componente. Toda huella local debe verificarse contra el dibujo
mecanico, numeracion de pines, pad 1 y recomendaciones de cobre del fabricante.

### Modelos 3D

Guardar modelos propios en `Libs/3D/` y enlazarlos desde la huella local. No
son necesarios para comenzar el esquema ni para validar el circuito electrico.

## Regla Antes De Reutilizar

Antes de copiar cualquier componente de la V1 se debe comprobar:

1. MPN, fabricante y estado de stock en JLCPCB.
2. Pinout del simbolo contra el datasheet del MPN exacto.
3. Pad numerado de la huella contra el pinout del datasheet.
4. Encapsulado, limites de tension, corriente y termica para la V2.

Copiar un dibujo del esquematico puede ser util para aprender una topologia;
no verifica que la libreria, el pinout o la huella sean correctos para fabricar.
