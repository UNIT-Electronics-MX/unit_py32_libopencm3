# PyOCD y programador CH552

[← Anterior: preparar el CH552](CH552_CMSIS_DAP.md) · **Ruta guiada: paso 4 de 7** · [Siguiente: preparar el reto →](RETO_FIRMWARE.md)

Aquí conocerás PyOCD, comprobarás que tu CH552 funcione como sonda CMSIS-DAP y aprenderás a conectarlo a una placa PY32F003 de práctica. Si el CH552 todavía necesita el firmware CMSIS-DAP, sigue la [guía para grabarlo](CH552_CMSIS_DAP.md). Si vas a cargar un ejemplo en el PY32F003, [compílalo primero](COMPILACION.md).

## Qué es PyOCD

**PyOCD** es un programa de tu laptop que se comunica con microcontroladores ARM mediante una sonda de depuración. Usarás el CH552 como esa sonda. Estas son las piezas de la práctica:

| Pieza | Papel |
| --- | --- |
| **PyOCD** | Programa instalado en `.venv` que envía comandos de lectura, grabación y reinicio |
| **CH552 con CMSIS-DAP** | Sonda USB que convierte esos comandos en señales SWD |
| **PY32F003** | Microcontrolador de práctica que recibe el firmware de ejemplo |
| **CMSIS-Pack local** | Describe a PyOCD el dispositivo `py32f003x4` |

**Ruta de la señal:** tu laptop con PyOCD → USB → CH552/CMSIS-DAP → SWD → PY32F003 de práctica.

> [!IMPORTANT]
> Hay **dos firmwares distintos**. `unit_ch55x_sdk/.../build/main.bin` pertenece al **CH552** y se carga con [chprog.py](CH552_CMSIS_DAP.md). `py32f003_lib/build/blink/blink.hex` o `uart.hex` pertenece al **PY32F003** y se graba en el [paso 4](FLASHEO.md).

## Verificar PyOCD y el soporte del PY32

Esta comprobación no necesita hardware. Activa el entorno virtual y consulta el pack incluido en el SDK:

```bash
cd ~/bugcon2026/py32f003_lib
source ~/bugcon2026/.venv/bin/activate
pyocd --version
pyocd list --targets --pack pyocd-packs/Puya.PY32.pack
```

La lista de objetivos debe incluir `py32f003x4`. Al configurar CMake para Blink o UART, el SDK genera `build/blink/pyocd.yaml` o `build/uart/pyocd.yaml`; esos archivos apuntan al mismo pack local. No necesitas descargarlos ni editarlos.

## Comprobar el CH552

Conecta **solo el CH552** al USB, sin mantener presionado `BOOT`:

```bash
source ~/bugcon2026/.venv/bin/activate
lsusb
pyocd list
```

En el programador usado para este taller se observó `ID 1a86:8011` con la descripción `MultiProtocol CMSIS-DAP`. Si `pyocd list` también muestra la sonda, **ya está lista**: continúa en [Conectar CH552 y PY32F003](#conectar-ch552-y-py32f003). `lsusb` por sí solo no confirma que PyOCD pueda acceder a ella.

Si `pyocd list` no muestra la sonda, consulta la [guía para cargar CMSIS-DAP en el CH552](CH552_CMSIS_DAP.md). Allí encontrarás los pasos de compilación, modo **BOOT**, grabación, permisos USB y verificación.

## Conectar CH552 y PY32F003

Con las dos placas desconectadas, revisa el [pinout del CH552](https://unit-electronics-mx.github.io/unit_devlab_ch552_multiprotocol_programmer_module/1_2_0_about.html) y une las señales **por nombre**:

| CH552 en modo SWD | Tarjeta PY32F003 |
| --- | --- |
| `SWCLK` o `SWC` | `SWCLK` / `PA14` |
| `SWDIO` o `SWD` | `SWDIO` / `PA13` |
| `GND` | `GND` |
| `VCC` | Alimentación o referencia de 3.3 V, según el montaje |

Selecciona **3.3 V** en el selector `JP2` del programador. Comprueba la polaridad antes de conectar `VCC`; si la tarjeta PY32 ya tiene alimentación propia, evita unir dos salidas de alimentación. El [arnés Qwiic de 4 pines](https://uelectronics.com/producto/arnes-qwiic-4-pines-pitch-1mm/) lleva las cuatro señales, pero verifica su orientación en **ambas** placas y que `VCC` no quede cruzado con `GND`.

Conecta la sonda al USB y ejecuta `pyocd list`. Este comando confirma que PyOCD ve **la sonda**; la comunicación con el **PY32** se comprobará al intentar [leer la memoria](LECTURA.md) o [grabar un ejemplo](FLASHEO.md).

## Siguiente paso

Si eres instructor y vas a preparar la placa, continúa con el [firmware didáctico del reto](RETO_FIRMWARE.md). Si la placa ya contiene el programa del reto, salta directamente a [leer su memoria](LECTURA.md). Para cargar Blink o UART en otra placa de práctica, consulta [flasheo](FLASHEO.md).
