# Instalación del entorno

[← Anterior: inicio](README.md) · **Ruta guiada: paso 1 de 7** · [Siguiente: compilación →](docs/COMPILACION.md)

Aquí prepararás tu **laptop**: instalarás las herramientas, descargarás el código y activarás un entorno Python. Podrás completar todo sin conectar las placas. [Consulta las actividades del taller](README.md#qué-harás-durante-el-taller).

> [!IMPORTANT]
> Descarga el repositorio y las dependencias antes del evento. La conexión a Internet en el lugar puede ser limitada.

## Requisitos

Computadora con Linux basado en `apt`:

| Sistema | Versiones | Arquitectura |
| --- | --- | --- |
| Debian | 12 o 13 | `amd64` o `arm64` |
| Ubuntu y derivadas, como Linux Mint o Pop!_OS | 22.04 LTS o 24.04 LTS | `amd64` o `arm64` |
| Raspberry Pi OS **64 bits** | Bookworm o posterior, en Raspberry Pi 4 o 5 | `arm64` |

> [!NOTE]
> En Raspberry Pi usa la edición de 64 bits: la de 32 bits (`armhf`) no tiene paquetes precompilados de `cmsis-pack-manager`, una dependencia de PyOCD, y su instalación con `pip` intentaría compilarla.

También necesitas acceso a `sudo`, Internet durante la instalación, al menos 5 GB libres, Python 3.9 o posterior y conocimientos básicos de terminal Linux y C. No se requiere un IDE. El día del taller lleva tu laptop con un puerto USB disponible; recibirás el kit en préstamo durante la sesión.

Comprueba tu sistema:

```bash
cat /etc/os-release
dpkg --print-architecture
python3 --version
```

`dpkg --print-architecture` debe mostrar `amd64` o `arm64`.

## 1. Instalar herramientas del sistema

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

El compilador ARM y CMake se usan en la [compilación](docs/COMPILACION.md). `libusb` y las herramientas Python sirven para [preparar el CH552](docs/CH552_CMSIS_DAP.md) y [usar PyOCD](docs/PYOCD.md). OpenOCD se instala como herramienta adicional; los ejemplos de esta ruta se graban en el PY32F003 con PyOCD.

Verifica las herramientas principales:

```bash
cmake --version
make --version
arm-none-eabi-gcc --version
arm-none-eabi-objcopy --version
gdb-multiarch --version
openocd --version
```

Cada comando debe mostrar una versión y terminar sin error.

## 2. Descargar el repositorio

El repositorio contiene dos submódulos: `py32f003_lib/` (firmware del PY32F003) y `unit_ch55x_sdk/` (firmware del programador CH552). El primero contiene a su vez la adaptación de `libopencm3`.

Clona en `~/bugcon2026`; esa carpeta no debe existir o debe estar vacía:

```bash
git clone --recurse-submodules https://github.com/UNIT-Electronics-MX/unit_py32_libopencm3.git ~/bugcon2026
cd ~/bugcon2026
git submodule status --recursive
```

> [!IMPORTANT]
> El ZIP de GitHub no incluye los submódulos. Si ya hiciste `git clone` sin `--recurse-submodules`, entra a `~/bugcon2026` y ejecuta `git submodule update --init --recursive`.

Comprueba que estén los archivos necesarios:

```bash
ls py32f003_lib/CMakeLists.txt py32f003_lib/pyocd-packs/Puya.PY32.pack
ls py32f003_lib/libopencm3/Makefile
ls unit_ch55x_sdk/examples/usb/prog/cmsisdap/Makefile
```

`git submodule status --recursive` debe mostrar los tres submódulos, incluido `py32f003_lib/libopencm3`. Un `-` al inicio indica que falta descargar uno; repite `git submodule update --init --recursive`. Un `+` indica que su versión difiere de la fijada por el repositorio principal.

### Actualizar antes del taller

```bash
cd ~/bugcon2026
git pull --recurse-submodules
git submodule sync --recursive
git submodule update --init --recursive
git submodule status --recursive
```

## 3. Preparar Python y PyOCD

PyOCD se instala en un entorno virtual dentro de la carpeta del taller; `.venv/` ya está excluida por `.gitignore`.

```bash
cd ~/bugcon2026
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
python -m pip install pyocd cmsis-pack-manager pyserial pyusb intelhex pipdeptree
```

Comprueba que Python y PyOCD salen del entorno virtual:

```bash
echo "$VIRTUAL_ENV"
command -v python
command -v pyocd
pyocd --version
python -m pip check
```

`command -v python` y `command -v pyocd` deben apuntar a `~/bugcon2026/.venv/bin/`. `python -m pip check` debe terminar sin dependencias incompatibles. En cada terminal nueva, activa el entorno otra vez:

```bash
cd ~/bugcon2026
source .venv/bin/activate
```

## Comprobación final

- [ ] Las herramientas del sistema muestran sus versiones.
- [ ] `git submodule status --recursive` no muestra un `-` inicial.
- [ ] Existen `py32f003_lib/CMakeLists.txt` y `py32f003_lib/libopencm3/Makefile`.
- [ ] `command -v pyocd` apunta a `.venv/bin/pyocd`.
- [ ] `python -m pip check` termina correctamente.

**Si quieres practicar con los ejemplos del SDK**, sigue la [guía de compilación de Blink o UART](docs/COMPILACION.md). Esa guía genera archivos en tu laptop; la grabación en una placa de práctica es un paso posterior.
