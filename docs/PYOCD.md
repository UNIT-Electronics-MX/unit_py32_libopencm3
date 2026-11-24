# PyOCD y programador CH552

[← Compilación](COMPILACION.md) · **Paso 3 de 4** · [Siguiente: flasheo →](FLASHEO.md)

Esta página explica la herramienta de grabación, comprueba que el CH552 funcione como sonda CMSIS-DAP y muestra cómo conectarlo al PY32F003. El firmware del PY32 ya debe estar [compilado](COMPILACION.md) antes de pasar al flasheo.

## Qué es PyOCD

**PyOCD** es un programa de la laptop que se comunica con microcontroladores ARM a través de una sonda de depuración. En este taller:

| Pieza | Papel |
| --- | --- |
| **PyOCD** | Programa instalado en `.venv` que envía comandos de grabación y reinicio |
| **CH552 con CMSIS-DAP** | Sonda USB que convierte esos comandos en señales SWD |
| **PY32F003** | Microcontrolador que recibe el firmware |
| **CMSIS-Pack local** | Describe a PyOCD el dispositivo `py32f003x4` |

**Ruta de la señal:** laptop con PyOCD → USB → CH552/CMSIS-DAP → SWD → PY32F003.

> [!IMPORTANT]
> Hay **dos firmwares distintos**. `unit_ch55x_sdk/.../build/main.bin` pertenece al **CH552** y solo se carga si la sonda necesita CMSIS-DAP. `py32f003_lib/build/blink/blink.hex` o `uart.hex` pertenece al **PY32F003** y se graba en el [paso 4](FLASHEO.md).

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

Si el CH552 aparece en `lsusb` como `1a86:8011` pero PyOCD informa falta de permisos, crea la regla udev para el **modo CMSIS-DAP**, vuelve a conectar la sonda y repite `pyocd list`:

```bash
echo 'SUBSYSTEMS=="usb", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="8011", MODE:="0666"' | sudo tee /etc/udev/rules.d/60-multiprotocol-cmsis-dap.rules
sudo udevadm control --reload-rules
```

El identificador `1a86:8011` es distinto de `4348:55e0`, que corresponde al cargador USB del CH552.

## Solo si hace falta: cargar CMSIS-DAP al CH552

Omite esta sección si `pyocd list` ya muestra la sonda. El [SDK CH55x de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk/tree/main/examples/usb/prog/cmsisdap) contiene el firmware CMSIS-DAP y su cargador USB. `sdcc` y `pyusb` se instalaron durante la [preparación del entorno](../installation.md).

Compila el firmware **del programador**:

```bash
cd ~/bugcon2026/unit_ch55x_sdk/examples/usb/prog/cmsisdap
make all
ls -lh build/main.bin
```

Desconecta el CH552. Mantén presionado `BOOT` mientras lo conectas de nuevo y luego suéltalo. `lsusb` debe mostrar el modo de arranque WCH `4348:55e0`. Con el entorno Python activo y desde la misma carpeta, carga el firmware:

```bash
source ~/bugcon2026/.venv/bin/activate
python3 tools/chprog.py build/main.bin
```

Si el cargador USB informa falta de permisos, aplica una regla **para el modo de arranque `4348:55e0`** y vuelve a conectarlo manteniendo `BOOT`:

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="4348", ATTR{idProduct}=="55e0", MODE="0666"' | sudo tee /etc/udev/rules.d/99-ch55x.rules
sudo udevadm control --reload-rules
```

Cuando termine, desconecta y reconecta el CH552 **sin** presionar `BOOT`. Comprueba otra vez `pyocd list`.

## Conectar CH552 y PY32F003

Con las dos placas desconectadas, revisa el [pinout del CH552](https://unit-electronics-mx.github.io/unit_devlab_ch552_multiprotocol_programmer_module/1_2_0_about.html) y une las señales **por nombre**:

| CH552 en modo SWD | Tarjeta PY32F003 |
| --- | --- |
| `SWCLK` o `SWC` | `SWCLK` / `PA14` |
| `SWDIO` o `SWD` | `SWDIO` / `PA13` |
| `GND` | `GND` |
| `VCC` | Alimentación o referencia de 3.3 V, según el montaje |

Selecciona **3.3 V** en el selector `JP2` del programador. Comprueba la polaridad antes de conectar `VCC`; si la tarjeta PY32 ya tiene alimentación propia, evita unir dos salidas de alimentación. El [arnés Qwiic de 4 pines](https://uelectronics.com/producto/arnes-qwiic-4-pines-pitch-1mm/) lleva las cuatro señales, pero verifica su orientación en **ambas** placas y que `VCC` no quede cruzado con `GND`.

Conecta la sonda al USB y ejecuta `pyocd list`. Este comando confirma que PyOCD ve **la sonda**; la comunicación con el **PY32** se confirmará al ejecutar el comando de [flasheo](FLASHEO.md).

## Siguiente paso

Con el `.hex` compilado, la sonda visible en `pyocd list` y las conexiones SWD revisadas, [graba el firmware en el PY32F003](FLASHEO.md).
