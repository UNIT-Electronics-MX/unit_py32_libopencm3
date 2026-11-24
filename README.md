# Del header al dump

### Hackeando embebidos con un programador de bolsillo

**BugCON Security Conference 2026** · Villa del Hardware de Electronic Cats<br>
**Jueves 26 de noviembre de 2026 · 10:00** · Instructor: César Bautista · Apoyo: Adrián Rabadán

En este taller crearás firmware para un **PY32F003** y lo grabarás mediante un **CH552 en modo CMSIS-DAP**. Los dos SDK están incluidos como submódulos de Git.

## Sigue este orden

| Paso | Guía | Qué consigues | ¿Necesitas las placas? |
| --- | --- | --- | --- |
| **1** | [Instalación](installation.md) | Herramientas, repositorio y entorno Python listos | No |
| **2** | [Compilación](docs/COMPILACION.md) | `blink.hex` o `uart.hex` creado en la laptop | No |
| **3** | [PyOCD y programador](docs/PYOCD.md) | Entiendes PyOCD, compruebas el CH552 y conectas SWD | Sí, para la conexión |
| **4** | [Flasheo](docs/FLASHEO.md) | Firmware grabado y ejecutándose en el PY32F003 | Sí |

**El flujo en una línea:** código C → CMake y compilador ARM → archivo `.hex` en la laptop → PyOCD → programador CH552 → memoria flash del PY32F003.

> [!NOTE]
> **Compilar** crea archivos en tu computadora. **Flashear** escribe uno de esos archivos en el microcontrolador. **PyOCD** es el programa que habla con la sonda CH552 para hacer esa grabación. [Lee la explicación completa](docs/PYOCD.md#qué-es-pyocd).

## Datos del taller

| | |
| --- | --- |
| **Evento** | [BugCON 2026](https://bugcon.org/), 26 y 27 de noviembre |
| **Lugar** | Villa del Hardware de Electronic Cats |
| **Horario** | Jueves 26 de noviembre, 10:00 |
| **Cupo** | 15 personas, por orden de llegada |
| **Nivel** | Intermedio |
| **Trae** | Laptop con Linux: Debian, Ubuntu o Raspberry Pi OS de 64 bits ([requisitos](installation.md#requisitos)) |
| **Patrocinan** | UNIT Electronics y LATE.AR |

## Material

| Material | Uso |
| --- | --- |
| [Tarjeta PY32F003L24D6TR UNIT DevLab](https://uelectronics.com/producto/tarjeta-de-desarrollo-py32f003l24d6tr-arm-32-bits-unit-devlab/) | Ejecuta Blink y los otros ejemplos |
| [Programador multiprotocolo CH552 UNIT DevLab](https://uelectronics.com/producto/ch552-usb-modulo-programador-multiprotocolo-swd-jtag-updi-usbasp-unit-devlab/) | Sirve de sonda CMSIS-DAP para grabar el PY32F003 |
| [Arnés Qwiic de 4 pines, pitch 1 mm](https://uelectronics.com/producto/arnes-qwiic-4-pines-pitch-1mm/) | Conecta `SWCLK`, `SWDIO`, `VCC` y `GND` entre las placas ([conexiones](docs/PYOCD.md#conectar-ch552-y-py32f003)) |

Puedes terminar [instalación](installation.md) y [compilación](docs/COMPILACION.md) antes del taller, sin hardware.

## Repositorio

| Ruta | Contenido |
| --- | --- |
| [`installation.md`](installation.md) | Preparación de la laptop y descarga del código |
| [`docs/COMPILACION.md`](docs/COMPILACION.md) | Biblioteca, CMake, selección de ejemplo y archivos generados |
| [`docs/PYOCD.md`](docs/PYOCD.md) | Qué es PyOCD, sonda CH552, firmware CMSIS-DAP y conexiones SWD |
| [`docs/FLASHEO.md`](docs/FLASHEO.md) | Comandos para grabar Blink o UART en el PY32F003 |
| [`py32f003_lib/`](https://github.com/Cesarbautista10/py32f003_lib) | SDK PY32F003, ejemplos y CMSIS-Pack local |
| `py32f003_lib/libopencm3/` | [Adaptación de libopencm3 para PY32](https://github.com/Cesarbautista10/libopencm3) |
| `unit_ch55x_sdk/` | [SDK CH55x de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk) y firmware CMSIS-DAP |

<details>
<summary>Para mantenedores: actualizar los submódulos</summary>

```bash
git submodule update --remote --recursive
git add py32f003_lib unit_ch55x_sdk
git commit -m "Actualizar submódulos"
```

`py32f003_lib` fija su propia versión de `libopencm3`; para cambiarla, actualízala en el repositorio del SDK y después actualiza aquí `py32f003_lib`.

</details>
