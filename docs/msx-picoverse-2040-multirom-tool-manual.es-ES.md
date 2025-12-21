# Proyecto MSX PicoVerse 2040

|PicoVerse Frontal|PicoVerse Trasera|
|---|---|
|![PicoVerse Frontal](/images/2025-12-02_20-05.png)|![PicoVerse Trasera](/images/2025-12-02_20-06.png)|

El PicoVerse 2040 es un cartucho para MSX basado en Raspberry Pi Pico que utiliza firmware intercambiable para ampliar las capacidades del equipo. Al cargar diferentes imágenes de firmware, el cartucho puede ejecutar juegos y aplicaciones MSX y emular dispositivos de hardware adicionales (como mappers de ROM, memoria RAM extra o interfaces de almacenamiento), añadiendo periféricos virtuales al MSX. Uno de esos firmwares es el sistema MultiROM, que proporciona un menú en pantalla para explorar y lanzar múltiples títulos ROM almacenados en el cartucho.

El cartucho también puede exponer el puerto USB‑C del Pico como dispositivo de almacenamiento masivo, permitiendo copiar ROMs, DSKs y otros archivos directamente desde un ordenador al cartucho.

Además, se puede usar una variante de firmware Nextor con soporte de mapper de memoria +240 KB para aprovechar completamente el hardware PicoVerse 2040 en sistemas MSX con memoria limitada.

Características disponibles en la versión actual del cartucho PicoVerse 2040:

* Sistema de menú MultiROM para seleccionar y lanzar ROMs MSX.
* Soporte para Nextor OS con mapper de memoria opcional de +240 KB.
* Soporte de dispositivo de almacenamiento masivo USB para cargar ROMs y DSKs.
* Soporte para varios mappers MSX (PL-16, PL-32, KonSCC, Linear, ASC-08, ASC-16, Konami, NEO-8, NEO-16).
* Compatibilidad con sistemas MSX, MSX2 y MSX2+.

## Manual del creador de UF2 MultiROM

Esta sección documenta la herramienta de consola `multirom` usada para generar imágenes UF2 (`multirom.uf2`) que programan el cartucho PicoVerse 2040.

Por defecto, la herramienta `multirom` escanea archivos ROM MSX en un directorio y los empaqueta en una sola imagen UF2 que puede flashearse al Raspberry Pi Pico. La imagen resultante suele contener el binario de firmware del Pico, la ROM del menú MultiROM, un área de configuración que describe cada entrada ROM y las cargas útiles de las ROMs.

Dependiendo de las opciones, `multirom` también puede producir UF2 que arrancan directamente en un firmware personalizado en lugar del menú MultiROM. Por ejemplo, se pueden generar UF2 con un firmware que implemente Nextor. En ese modo, el puerto USB‑C del Pico puede usarse como almacenamiento masivo para cargar ROMs y DSKs desde Nextor (por ejemplo, mediante SofaRun).

## Visión general

Si se ejecuta sin opciones, la herramienta `multirom.exe` escanea el directorio de trabajo actual en busca de archivos ROM MSX (`.ROM` o `.rom`), analiza cada ROM para intentar adivinar el tipo de mapper, construye una tabla de configuración describiendo cada ROM (nombre, byte de mapper, tamaño, offset) e inserta esta tabla en una porción de la ROM de menú MSX. La herramienta concatena el blob de firmware del Pico, la porción de la ROM de menú, el área de configuración y las ROMs, y serializa toda la imagen en un archivo UF2 llamado `multirom.uf2`.

La copia UF2 (normalmente `multirom.uf2`) puede copiarse al dispositivo de almacenamiento masivo USB del Pico para flashear la imagen combinada. Hay que conectar el Pico manteniendo pulsado el botón BOOTSEL para entrar en el modo UF2. Aparecerá una nueva unidad USB llamada `RPI-RP2` y se puede copiar `multirom.uf2` a ella. Tras completarse la copia, desconecte el Pico e inserte el cartucho en su MSX para arrancar el menú MultiROM.

Los nombres de archivo ROM se usan para nombrar las entradas en el menú MSX. Hay un límite de 50 caracteres por nombre. Se muestra un efecto desplazable para nombres más largos, pero si el nombre supera 50 caracteres se truncará.

Si desea usar Nextor con su PicoVerse 2040 debe ejecutar la herramienta con la opción `-s1` o `-s2` para incluir la ROM Nextor embebida en la imagen. La opción `-s1` incluye la ROM Nextor estándar sin soporte de mapper, mientras que `-s2` incluye una ROM Nextor con soporte de mapper +240 KB. La ROM Nextor embebida será el único firmware cargado al arrancar y el menú MultiROM no estará disponible. A continuación puede usar SofaRun para cargar ROMs y DSKs desde el almacenamiento masivo USB del Pico.

Nota importante: Para usar una unidad USB normal puede necesitar un adaptador OTG o un cable para convertir el puerto USB‑C a USB‑A hembra.

## Uso por línea de comandos

Solo se proporcionan ejecutables de Microsoft Windows por ahora (`multirom.exe`).

### Uso básico:

```
multirom.exe [options]
```

### Opciones:
- `-n`, `--nextor`: Incluye la ROM NEXTOR embebida desde la configuración y la salida. Esta opción aún es experimental y por ahora solo funciona en modelos específicos MSX2.
- `-h`, `--help`: Muestra la ayuda y sale.
- `-o <filename>`, `--output <filename>`: Establece el nombre del archivo UF2 de salida (por defecto `multirom.uf2`).
- `-s1`: El cartucho arranca con Sunrise IDE Nextor (sin mapper de memoria).
- `-s2`: El cartucho arranca con Sunrise IDE Nextor (+240 KB de mapper de memoria).
- Si no se especifica `-s1` ni `-s2`, la herramienta produce una imagen MultiROM con el menú.
- Si necesita forzar un tipo de mapper específico para una ROM, puede añadir una etiqueta de mapper antes de la extensión `.ROM` en el nombre del archivo. La etiqueta no distingue mayúsculas de minúsculas. Por ejemplo, nombrar un archivo `Knight Mare.PL-32.ROM` fuerza el uso del mapper PL-32 para esa ROM. Las etiquetas como `SYSTEM` se ignoran.

### Ejemplos
- Produce `multirom.uf2` con el menú MultiROM y todas las `.ROM` en el directorio actual:
```
multirom.exe
```

- Produce `multirom.uf2` con la ROM Sunrise IDE Nextor (sin mapper de memoria):
```
multirom.exe -s1
```
- Produce `multirom.uf2` con la ROM Sunrise IDE Nextor (+240 KB de mapper de memoria):
```
multirom.exe -s2
```

## Cómo funciona (alto nivel)

1. La herramienta escanea el directorio de trabajo actual buscando archivos con extensión `.ROM` o `.rom`. Para cada archivo:
   - Extrae un nombre para mostrar (nombre de archivo sin extensión, truncado a 50 caracteres).
   - Obtiene el tamaño del archivo y valida que esté entre `MIN_ROM_SIZE` y `MAX_ROM_SIZE`.
   - Llama a `detect_rom_type()` para determinar heurísticamente el byte de mapper a usar en la entrada de configuración. Si hay una etiqueta de mapper en el nombre del archivo, esta anula la detección.
   - Si la detección falla, el archivo se omite.
   - Serializa el registro de configuración por ROM (nombre de 50 bytes, 1 byte de mapper, 4 bytes de tamaño LE, 4 bytes de offset en flash LE) en el área de configuración.
2. Tras el escaneo, la herramienta concatena (en orden): el binario de firmware del Pico embebido, una porción inicial de la ROM del menú MSX (`MENU_COPY_SIZE` bytes), el área completa de configuración (`CONFIG_AREA_SIZE` bytes), la ROM NEXTOR opcional y luego las cargas útiles ROM descubiertas en orden de descubrimiento.
3. La carga combinada se escribe como un archivo UF2 llamado `multirom.uf2` usando `create_uf2_file()` que produce bloques UF2 de 256 bytes dirigidos a la dirección de flash del Pico `0x10000000`.

## Heurísticas de detección de mappers
- `detect_rom_type()` implementa una combinación de comprobaciones de firma (cabecera "AB", etiquetas `ROM_NEO8` / `ROM_NE16`) y escaneo heurístico de opcodes y direcciones para elegir mappers MSX comunes, incluyendo (pero no limitado a):
  - Plain 16KB (mapper byte 1) — comprobación de cabecera 16KB AB
  - Plain 32KB (mapper byte 2) — comprobación de cabecera 32KB AB
  - Linear0 mapper (mapper byte 4) — comprobación especial de layout AB
  - NEO8 (mapper byte 8) y NEO16 (mapper byte 9)
  - Konami, Konami SCC, ASCII8, ASCII16 y otros mediante puntuación ponderada
- Si no se puede detectar un mapper con fiabilidad, la herramienta omite la ROM y reporta "unsupported mapper". Puede forzar un mapper mediante la etiqueta en el nombre del archivo.
- Solo los siguientes mappers son compatibles en el área de configuración y el menú: `PL-16,  PL-32,  KonSCC,  Linear,  ASC-08,  ASC-16,  Konami,  NEO-8,  NEO-16`

## Uso del selector de ROM MSX (menú)

Al encender el MSX con el cartucho PicoVerse 2040 insertado, aparece el menú MultiROM mostrando la lista de ROMs disponibles. Puede navegar con las teclas de flecha del teclado.

Use las flechas Arriba y Abajo para mover el cursor de selección por la lista de ROMs. Si tiene más de 19 ROMs, use las flechas Izquierda y Derecha para desplazarse por las páginas de entradas.

Pulse Enter o Espacio para ejecutar la ROM seleccionada. El MSX intentará arrancar la ROM usando la configuración de mapper adecuada.

En cualquier momento dentro del menú, puede pulsar la tecla H para ver la pantalla de ayuda con instrucciones básicas. Pulse cualquier tecla para volver al menú principal.

## Uso de Nextor con el cartucho PicoVerse 2040

Si flasheó el cartucho con firmware Nextor (usando `-s1` o `-s2`), el cartucho arrancará directamente en Nextor en lugar del menú MultiROM. Esas opciones se comunican con el MSX usando E/S mapeadas en memoria, por lo que el cartucho debe insertarse en una ranura de cartucho estándar.

Una vez Nextor esté en ejecución, puede usar SofaRun u otro lanzador compatible con Nextor para cargar ROMs y DSKs desde el almacenamiento masivo USB del Pico. Puede necesitar un adaptador o cable OTG para conectar pendrives USB-A estándar al puerto USB‑C del cartucho.

En sistemas MSX2 o MSX2+ con la versión Nextor +240 KB (opción S2), el cartucho añadirá la memoria adicional al sistema y la secuencia de arranque del BIOS reflejará el aumento de RAM. Algunos modelos MSX muestran la memoria expandida total durante el arranque.

La opción `-n` de la herramienta multirom incluye una ROM Nextor experimental embebida que puede usarse en modelos MSX2 específicos. Sin embargo, esta opción aún está en desarrollo y puede no funcionar en todos los sistemas. Esta versión usa un método distinto (basado en puertos IO) para comunicarse con el MSX, permitiendo su uso en sistemas donde la ranura de cartucho no es estándar/expandida.

Más detalles sobre el protocolo usado para comunicarse con el ordenador MSX están en Nextor-Pico-Bridge-Protocol.md. Los detalles de la comunicación con el firmware Sunrise IDE Nextor están en Sunrise-Nextor.md.

### Cómo preparar una unidad USB para Nextor

1. Conecte la unidad USB a su PC.
2. Cree una partición FAT16 de máximo 4GB en la unidad USB. Puede usar herramientas del sistema operativo o software de particionado de terceros.
3. Copie los archivos del sistema Nextor al directorio raíz de la partición FAT16. Puede obtener los archivos de Nextor desde la distribución oficial o repositorio. También necesita el archivo COMMAND2 para la shell de comandos:
   1. [Nextor Download Page](https://github.com/Konamiman/Nextor/releases)
   2. [Command2 Download Page](http://www.tni.nl/products/command2.html)
4. Copie cualquier ROM MSX (`.ROM`) o imágenes de disco (`.DSK`) que quiera usar al directorio raíz de la unidad.
5. Instale SofaRun u otro lanzador compatible con Nextor en la unidad si planea usarla para lanzar ROMs y DSKs. Puede descargar SofaRun desde: [SofaRun](https://www.louthrax.net/mgr/sofarun.html)
6. Expulse la unidad USB de forma segura.

## Ideas de mejora
- Mejorar las heurísticas de detección de ROM para cubrir más mappers y casos límite.
- Implementar pantalla de configuración para cada entrada ROM (nombre, anulación de mapper, etc).
- Añadir soporte para más mappers de ROM.
- Implementar un menú gráfico con mejor navegación e información de ROM.
- Añadir soporte para guardar/cargar la configuración del menú para preservar preferencias.
- Soportar archivos DSK también, con entradas de configuración adecuadas.
- Mejorar la detección de mappers para cubrir más casos.
- Añadir soporte para slices de ROM de menú o temas personalizados.
- Permitir descargar ROMs desde URLs e incrustarlas directamente.
- Soporte inalámbrico (wifi) mediante módulo ESP-01 externo.
- Permitir el uso del joystick para navegar el menú.
- Soporte para comprimir ROMs para ahorrar espacio.

## Problemas conocidos

- La inclusión de la ROM Nextor embebida sigue siendo experimental y puede no funcionar en todos los modelos MSX2.
- Algunas ROMs con mappers poco comunes pueden no detectarse correctamente y se omitirán a menos que se fuerce un mapper válido mediante etiqueta.
- Actualmente la herramienta solo soporta ejecutables de Windows. No hay versiones para Linux o macOS.
- La herramienta no valida la integridad de las ROMs más allá de tamaño y comprobaciones de cabecera básicas. ROMs corruptas pueden causar comportamientos inesperados.
- El menú MultiROM no soporta archivos DSK; solo se listan y ejecutan ROMs.
- La herramienta no soporta subdirectorios; solo procesa ROMs en el directorio de trabajo actual.
- La memoria flash del Pico puede desgastarse tras muchos ciclos de escritura. Evite re‑flasheos excesivos.

## Modelos MSX probados

| Modelo | Tipo | Estado | Comentarios |
| --- | --- | --- | --- |
| Adermir Carchano Expert 4 | MSX2+ | OK | Verificado |
| Gradiente Expert | MSX1 | OK | Verificado |
| JFF MSX | MSX1 | OK | Verificado |
| MSX Book | MSX2+ (clon FPGA) | OK | Verificado |
| MSX One | MSX1 | Not OK | Cartucho no reconocido |
| National FS-4500 | MSX1 | OK | Verificado |
| Panasonic FS-A1GT | TurboR | OK | Verificado |
| Panasonic FS-A1ST | TurboR | OK | Verificado |
| Panasonic FS-A1WX | MSX2+ | OK | Verificado |
| Panasonic FS-A1WSX | MSX2+ | OK | Verificado |
| Sharp HotBit HB8000 | MSX1 | OK | Verificado |
| SMX-HB | MSX2+ (clon FPGA) | OK | Verificado |
| Sony HB-F1XD | MSX2 | OK | Verificado |
| Sony HB-F1XDJ | MSX2 | OK | Verificado |
| Sony HB-F1XV | MSX2+ | OK | Verificado |
| TRHMSX | MSX2+ (clon FPGA) | OK | Verificado |
| uMSX | MSX2+ (clon FPGA) | OK | Verificado |
| Yamaha YIS604 | MSX1 | OK | Verificado |

La versión Sunrise Nextor +240K ha sido probada en:

| Modelo | Estado | Comentarios |
| --- | --- | --- |
| MSX1 | | |
| Sharp HotBit HB8000 (MSX1) | Not OK | El mapper de memoria no funciona y Nextor no es funcional |
| MSX2+ | | |
| TRHMSX (MSX2+, clon FPGA) | OK | Verificado |
| uMSX (MSX2+, clon FPGA) | OK | Verificado |

Autor: Cristiano Almeida Goncalves
Última actualización: 12/20/2025
