# Del header al dump: instalación del entorno

**BugCON Security Conference 2026 · Villa del Hardware**  
**Instructor:** César Bautista · **Fecha:** jueves 26 de noviembre de 2026

Este manual permite preparar y comprobar el entorno de desarrollo, configurar el programador CH552 en modo CMSIS-DAP y grabar Blink en la tarjeta PY32F003. Las secciones de instalación y compilación pueden completarse sin hardware. Conviene descargar los repositorios antes del evento, ya que la conexión a Internet en el lugar puede ser limitada.

## Requisitos

- Computadora con Debian 12 o 13, o una distribución derivada compatible con `apt` (arquitectura `amd64` recomendada).
- Acceso a `sudo`, conexión a Internet durante la instalación y espacio libre para el compilador y las dependencias (se recomiendan al menos 5 GB).
- Python 3.9 o posterior y un puerto USB disponible para el día del taller.
- Conocimientos básicos de terminal Linux y programación en C. No se requiere un IDE.

Comprueba tu sistema:

```bash
cat /etc/os-release
dpkg --print-architecture
python3 --version
```

## 1. Instalar las herramientas del sistema

```bash
sudo apt update
sudo apt install -y \
  git cmake make build-essential binutils sdcc \
  gcc-arm-none-eabi binutils-arm-none-eabi libnewlib-arm-none-eabi \
  gdb-multiarch openocd \
  python3 python3-pip python3-venv \
  libusb-1.0-0 usbutils picocom \
  file xxd
```

Estas herramientas cubren compilación ARM, depuración, comunicación USB y revisión de archivos. `picocom` se utilizará cuando corresponda trabajar con una interfaz serial.

Comprueba las instalaciones principales:

```bash
cmake --version
make --version
arm-none-eabi-gcc --version
arm-none-eabi-objcopy --version
gdb-multiarch --version
openocd --version
```

Cada comando debe mostrar una versión y terminar sin error. OpenOCD se verifica aquí como herramienta instalada; su soporte específico para el PY32F003 depende de la configuración usada durante el taller.

## 2. Descargar el repositorio del taller

El [repositorio del taller](https://github.com/UNIT-Electronics-MX/unit_py32_libopencm3-) reúne todo el código como submódulos de Git, así que un solo `git clone` deja todo listo:

| Ruta | Contenido |
| --- | --- |
| `py32f003_lib/` | [SDK del PY32F003](https://github.com/Cesarbautista10/py32f003_lib): proyecto CMake, ejemplos y CMSIS-Pack local |
| `py32f003_lib/libopencm3/` | [Adaptación de libopencm3 para PY32](https://github.com/Cesarbautista10/libopencm3) (submódulo del SDK) |
| `unit_ch55x_sdk/` | [SDK CH55x de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk): firmware CMSIS-DAP y cargador USB del programador CH552 |
| `installation.md` | Este manual |

### 2.1. Clonar con todos los submódulos

Clona directamente en `~/bugcon2026`; la carpeta no debe existir o debe estar vacía:

```bash
git clone --recurse-submodules https://github.com/UNIT-Electronics-MX/unit_py32_libopencm3-.git ~/bugcon2026
cd ~/bugcon2026
```

`--recurse-submodules` descarga también los submódulos anidados, como `libopencm3` dentro del SDK.

### 2.2. Si clonaste sin `--recurse-submodules`

Las carpetas de los submódulos existen pero están vacías. Complétalas desde la raíz del repositorio:

```bash
cd ~/bugcon2026
git submodule update --init --recursive
```

### 2.3. No uses "Download ZIP"

El botón **Code → Download ZIP** de GitHub **no incluye submódulos**: `py32f003_lib`, `libopencm3` y `unit_ch55x_sdk` quedan vacías y, sin la carpeta `.git`, `git submodule` no puede completarlas. Si descargaste el ZIP, bórralo y sigue la sección 2.1.

### 2.4. Comprobar la descarga

```bash
cd ~/bugcon2026
git submodule status --recursive
ls py32f003_lib/CMakeLists.txt py32f003_lib/pyocd-packs/Puya.PY32.pack
ls py32f003_lib/libopencm3/Makefile
ls unit_ch55x_sdk/examples/usb/prog/cmsisdap/Makefile
```

`git submodule status` debe listar `py32f003_lib`, `py32f003_lib/libopencm3` y `unit_ch55x_sdk`. Un `-` al inicio de una línea indica que ese submódulo no se descargó; repite la sección 2.2. Un `+` indica que su versión no coincide con la del repositorio del taller.

El SDK usa el identificador de dispositivo `py32f003x4`. Su CMSIS-Pack local evita tener que descargar un pack adicional para consultar su soporte con PyOCD.

### 2.5. Actualizar antes del taller

Si el repositorio recibe cambios antes del evento, actualiza todo con:

```bash
cd ~/bugcon2026
git pull
git submodule sync --recursive
git submodule update --init --recursive
```

## 3. Preparar el entorno Python

Mantén las herramientas Python en un entorno virtual dentro de la carpeta del taller, sin modificar los paquetes del sistema (`.gitignore` ya excluye `.venv`):

```bash
cd ~/bugcon2026
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install pyocd cmsis-pack-manager pyserial pyusb intelhex pipdeptree
```

Verifica que se esté usando el entorno virtual y que las dependencias sean coherentes:

```bash
echo "$VIRTUAL_ENV"
command -v python
command -v pyocd
pyocd --version
pyocd pack --help
python -m pip check
```

`command -v python` y `command -v pyocd` deben apuntar a `~/bugcon2026/.venv/bin/`; `python -m pip check` debe indicar que no hay dependencias incompatibles. En una terminal nueva, vuelve a activarlo con:

```bash
cd ~/bugcon2026
source .venv/bin/activate
```

## 4. Compilar la biblioteca y Blink

Empieza siempre desde la raíz de `py32f003_lib`, sin importar el directorio en el que esté abierta la terminal. Compila la biblioteca con `make -C`, que ejecuta Make dentro de `libopencm3` sin cambiar el directorio actual:

```bash
cd ~/bugcon2026/py32f003_lib
ls CMakeLists.txt libopencm3/Makefile
make -C libopencm3 TARGETS=py32/f0
ls -lh libopencm3/lib/libopencm3_py32f0.a
```

Ejecuta el siguiente bloque solo si el anterior terminó sin errores. `CMakeLists.txt` está en `py32f003_lib`; desde esa raíz, configura y compila Blink:

```bash
cmake -S . -B build/blink \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake \
  -DAPP=blink
cmake --build build/blink -j
```

Comprueba los resultados:

```bash
ls -lh build/blink/blink.elf build/blink/blink.bin build/blink/blink.hex
ls -lh build/blink/blink.map build/blink/generated.py32f003x4.ld build/blink/pyocd.yaml
arm-none-eabi-size build/blink/blink.elf
```

Los archivos `.elf`, `.bin` y `.hex` deben existir. La configuración `pyocd.yaml` se genera durante el paso de CMake y apunta al pack incluido en el repositorio.

## 5. Verificar el soporte de PyOCD

Con el entorno virtual activo y desde la raíz del SDK:

```bash
pyocd list --targets --pack pyocd-packs/Puya.PY32.pack
```

Confirma que la lista contiene `py32f003x4`. Esta comprobación usa el pack local y no requiere conectar hardware.

## 6. Configurar el programador CH552 multiprotocolo

Para trabajar con el PY32F003, el [programador multiprotocolo CH552 de UNIT](https://github.com/UNIT-Electronics-MX/unit_devlab_ch552_multiprotocol_programmer_module) debe ejecutar el firmware **CMSIS-DAP para ARM/SWD**. Los firmwares para AVR o CPLD corresponden a otros protocolos y PyOCD no los detectará como una sonda CMSIS-DAP.

### 6.1. Comprobar el modo actual

Conecta solo el CH552 al USB, sin mantener presionado `BOOT`, y ejecuta:

```bash
source ~/bugcon2026/.venv/bin/activate
lsusb
pyocd list
```

En el programador usado para este taller se observó esta salida de `lsusb`:

```text
Bus 005 Device 003: ID 1a86:8011 QinHeng Electronics MultiProtocol CMSIS-DAP
```

El identificador `1a86:8011` y la descripción `MultiProtocol CMSIS-DAP` indican que el adaptador USB **ya está en modo CMSIS-DAP**. Si `pyocd list` también muestra la sonda, continúa con las conexiones SWD; no hace falta volver a cargar el firmware del programador. `lsusb` por sí solo no confirma el acceso desde PyOCD ni la comunicación con el PY32.

Si aparece en `lsusb` pero `pyocd list` no lo encuentra o informa falta de permisos, configura una regla udev para **el modo CMSIS-DAP `1a86:8011`**, vuelve a conectar el dispositivo y repite `pyocd list`:

```bash
echo 'SUBSYSTEMS=="usb", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="8011", MODE:="0666"' | sudo tee /etc/udev/rules.d/60-multiprotocol-cmsis-dap.rules
sudo udevadm control --reload-rules
```

Este identificador es distinto del `4348:55e0` del cargador USB del CH552. PyOCD [incluye soporte para sondas WCH-Link con `1a86:8011`](https://github.com/pyocd/pyOCD#requirements); por eso conviene resolver primero el acceso USB y verificar la versión de PyOCD si la sonda aún no aparece.

### 6.2. Compilar y cargar CMSIS-DAP en el CH552

Sigue esta subsección únicamente si el programador no tiene firmware CMSIS-DAP. El [SDK CH55x de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk/tree/main/examples/usb/prog/cmsisdap) contiene el código y el cargador USB para esta placa. El entorno Python creado en la sección 3 ya incluye `pyusb`.

El SDK ya se descargó con el repositorio del taller (sección 2):

```bash
cd ~/bugcon2026/unit_ch55x_sdk/examples/usb/prog/cmsisdap
make all
ls -lh build/main.bin
```

`build/main.bin` es el firmware **del CH552**. Para entrar en el cargador USB, desconecta el programador, mantén presionado su botón `BOOT` mientras lo conectas y luego suelta el botón. Comprueba que `lsusb` muestre el dispositivo de arranque WCH (`4348:55e0`):

```bash
lsusb
```

Con el entorno virtual activo y desde `examples/usb/prog/cmsisdap`, cárgalo al CH552:

```bash
python3 tools/chprog.py build/main.bin
```

El programa debe indicar que escribió y verificó los bytes. Si aparece un error de acceso USB en Debian, configura una regla para **el dispositivo de arranque** CH55x y vuelve a conectarlo manteniendo `BOOT`:

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="4348", ATTR{idProduct}=="55e0", MODE="0666"' | sudo tee /etc/udev/rules.d/99-ch55x.rules
sudo udevadm control --reload-rules
```

Después de cargar el firmware, desconecta el CH552 y conéctalo de nuevo **sin** presionar `BOOT`. Ejecuta `pyocd list` y confirma que aparezca la sonda CMSIS-DAP. Si el sistema la ve por USB pero PyOCD informa falta de permisos, aplica la regla para el identificador que muestre `lsusb` en modo normal.

### 6.3. Conectar CH552 y PY32F003

Con ambos dispositivos desconectados, revisa el [pinout del CH552](https://unit-electronics-mx.github.io/unit_devlab_ch552_multiprotocol_programmer_module/1_2_0_about.html) y une las señales por nombre:

| CH552 en modo SWD | Tarjeta PY32F003 |
| --- | --- |
| `SWCLK` o `SWC` | `SWCLK` / `PA14` |
| `SWDIO` o `SWD` | `SWDIO` / `PA13` |
| `GND` | `GND` |
| `VCC` | Alimentación o referencia de 3.3 V, según el montaje |

Selecciona **3.3 V** en el selector de tensión `JP2` del programador. Comprueba la polaridad y la tensión antes de conectar `VCC`; si la tarjeta PY32 ya tiene alimentación propia, evita unir dos salidas de alimentación. El conector JST de cuatro vías reúne `SWCLK`, `SWDIO`, `VCC` y `GND`, pero verifica su orientación física en ambas placas antes de insertarlo.

## 7. Grabar `blink.bin` en el PY32F003

Vuelve a la raíz del SDK PY32 y comprueba que el CH552 aparezca en PyOCD:

```bash
cd ~/bugcon2026/py32f003_lib
source ~/bugcon2026/.venv/bin/activate
pyocd list
```

El firmware de Blink **es para el PY32F003**, no para el CH552. Grábalo con cualquiera de las dos opciones siguientes; ambas usan el CMSIS-Pack local indicado por `build/blink/pyocd.yaml` y reinician el PY32 al terminar.

### 7.1. Opción A: objetivo de CMake (recomendada)

Desde la raíz de `py32f003_lib`, con Blink ya compilado (sección 4):

```bash
cmake --build build/blink --target flash-reset -j
```

Este objetivo graba `build/blink/blink.hex`, que ya incluye la dirección de carga, y reinicia la tarjeta. El objetivo `flash` graba sin reiniciar. Escribe `--build` y `--target` tal cual; un error como `cmake --hash` produce `Unknown argument`.

### 7.2. Opción B: PyOCD directo con el `.bin`

Como un archivo `.bin` no incluye dirección de carga, indica el inicio de la Flash del PY32 (`0x08000000`) al grabarlo:

```bash
pyocd flash \
  -t py32f003x4 \
  --config build/blink/pyocd.yaml \
  --base-address 0x08000000 \
  build/blink/blink.bin
```

### 7.3. Comprobar el resultado

El comando debe terminar sin errores y el LED de Blink debe empezar a parpadear. Para grabar otro ejemplo, cambia `blink` por su nombre en `-DAPP=` y en la ruta `build/<ejemplo>`; por ejemplo, `cmake --build build/uart --target flash-reset -j`.

Si PyOCD no encuentra la sonda, revisa el firmware CMSIS-DAP y los permisos USB del CH552. Si encuentra la sonda pero no el PY32, revisa `SWDIO`, `SWCLK`, `GND`, la alimentación de 3.3 V y el identificador `py32f003x4`.

## Lista de verificación

Los últimos tres puntos requieren disponer de las dos placas; pueden completarse durante el taller.

- [ ] `apt` instaló las herramientas sin errores.
- [ ] El repositorio del taller está en `~/bugcon2026` y `git submodule status --recursive` no muestra `-`.
- [ ] `cmake`, `make`, el compilador ARM, GDB y OpenOCD muestran su versión.
- [ ] El entorno virtual está activo y `python -m pip check` termina correctamente.
- [ ] PyOCD muestra su versión y el subcomando `pack` está disponible.
- [ ] Existe `libopencm3/lib/libopencm3_py32f0.a`.
- [ ] Blink se compiló y generó archivos `.elf`, `.bin` y `.hex`.
- [ ] PyOCD muestra `py32f003x4` al consultar el pack local.
- [ ] El CH552 ejecuta CMSIS-DAP y aparece en `pyocd list`.
- [ ] Las conexiones SWD y la tensión de 3.3 V están verificadas.
- [ ] `blink.bin` se grabó en el PY32F003 y el LED responde.

Si falla algún paso, conserva el mensaje de error completo y el comando ejecutado para revisarlos en el taller.

## Referencias

- [Repositorio del taller](https://github.com/UNIT-Electronics-MX/unit_py32_libopencm3-)
- [SDK PY32F003 y guía de compilación](https://github.com/Cesarbautista10/py32f003_lib)
- [Documentación de instalación de PyOCD](https://pyocd.io/docs/installing.html)
- [Soporte de dispositivos y CMSIS-Packs de PyOCD](https://pyocd.io/docs/target_support.html)
- [Documentación de UNIT DevLab PY32F003](https://wiki.uelectronics.com/wiki/devlab_py32f003l24d6tr_mcu_dev_board)
- [Programador CH552 multiprotocolo de UNIT](https://github.com/UNIT-Electronics-MX/unit_devlab_ch552_multiprotocol_programmer_module)
- [Firmware CMSIS-DAP para CH552 de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk/tree/main/examples/usb/prog/cmsisdap)
- [Comando de carga de PyOCD](https://github.com/pyocd/pyOCD/blob/main/pyocd/subcommands/load_cmd.py)
