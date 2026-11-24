# Del header al dump: taller PY32F003

**BugCON Security Conference 2026 · Villa del Hardware**
**Instructor:** César Bautista · **Fecha:** jueves 26 de noviembre de 2026

Este repositorio reúne todo el código del taller como submódulos de Git: el SDK del microcontrolador PY32F003 (con su adaptación de libopencm3) y el SDK del programador CH552.

## Descarga rápida

```bash
git clone --recurse-submodules https://github.com/UNIT-Electronics-MX/unit_py32_libopencm3.git ~/bugcon2026
cd ~/bugcon2026
git submodule status --recursive
```

> **No uses "Code → Download ZIP".** El ZIP de GitHub no incluye submódulos y las carpetas quedarían vacías.

Si ya clonaste sin `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

Para actualizar antes del taller:

```bash
git pull
git submodule sync --recursive
git submodule update --init --recursive
```

## Contenido

| Ruta | Contenido |
| --- | --- |
| [`installation.md`](installation.md) | Manual paso a paso: herramientas, entorno Python, compilación y grabación de Blink |
| `py32f003_lib/` | [SDK del PY32F003](https://github.com/Cesarbautista10/py32f003_lib): proyecto CMake, ejemplos (blink, uart, pwm, adc) y CMSIS-Pack local |
| `py32f003_lib/libopencm3/` | [libopencm3 adaptada a PY32](https://github.com/Cesarbautista10/libopencm3), submódulo del SDK |
| `unit_ch55x_sdk/` | [SDK CH55x de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk): firmware CMSIS-DAP y cargador USB del programador CH552 |

## Resumen del flujo

Sigue [installation.md](installation.md) para el detalle. Con las herramientas y el entorno Python ya instalados:

```bash
cd ~/bugcon2026
source .venv/bin/activate
cd py32f003_lib

# Compilar libopencm3 (una vez) y Blink
make -C libopencm3 TARGETS=py32/f0
cmake -S . -B build/blink -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake -DAPP=blink
cmake --build build/blink -j

# Grabar en el PY32F003 con el CH552 en modo CMSIS-DAP
pyocd list
cmake --build build/blink --target flash-reset -j
```

## Para mantenedores

Las versiones de los submódulos quedan fijadas en este repositorio. Para pasar a la última versión de cada uno:

```bash
git submodule update --remote --recursive
git add py32f003_lib unit_ch55x_sdk
git commit -m "Actualizar submódulos"
```

`py32f003_lib` fija su propia versión de `libopencm3`; para cambiarla, actualízala en el repositorio del SDK y después actualiza aquí `py32f003_lib`.
