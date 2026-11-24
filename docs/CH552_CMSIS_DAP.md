# Cargar CMSIS-DAP en el CH552

[← Instalación](../installation.md) · [Volver al inicio](../README.md) · [Siguiente: PyOCD y conexión SWD →](PYOCD.md)

El **CH552** es la sonda USB que conectarás a la laptop. Para que PyOCD pueda usarlo, debe ejecutar el firmware **CMSIS-DAP** del [SDK CH55x de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk/tree/main/examples/usb/prog/cmsisdap). Esta guía muestra cómo compilar ese firmware y grabarlo **en el CH552**. El archivo resultante es `main.bin`; los ejemplos `blink.hex` y `uart.hex` de la [guía de compilación](COMPILACION.md) se destinan a la placa PY32F003.

La grabación sustituye el programa que tenga actualmente el CH552. Si `pyocd list` ya lo muestra como sonda CMSIS-DAP, está listo y puedes pasar a [conectarlo a la placa](PYOCD.md#conectar-ch552-y-py32f003).

## 1. Preparar las herramientas

Completa primero la [instalación](../installation.md): necesitas `sdcc`, `make`, `objcopy` y `pyusb`. En una terminal, activa el entorno Python y comprueba `pyusb`:

```bash
cd ~/bugcon2026
source .venv/bin/activate
python -c "import usb; print('PyUSB disponible')"
```

Desconecta la placa PY32F003: durante estos pasos solo conectarás el **CH552** al USB.

## 2. Compilar el firmware del CH552

```bash
cd ~/bugcon2026/unit_ch55x_sdk/examples/usb/prog/cmsisdap
make all
ls -lh build/main.bin
```

Si `make all` termina sin errores y existe `build/main.bin`, tienes el archivo que se grabará en el CH552. Puedes compilarlo antes de conectar la sonda.

## 3. Entrar en el modo de arranque

1. Desconecta el CH552 del USB.
2. Mantén presionado el botón **BOOT** del CH552.
3. Sin soltar **BOOT**, conecta el CH552 al USB. Después suelta el botón.
4. Comprueba que aparece el cargador USB:

```bash
lsusb -d 4348:55e0
```

Debe aparecer un dispositivo con identificador `4348:55e0`. Si no aparece, repite la conexión manteniendo **BOOT** desde antes de insertar el USB. El modo de arranque puede durar poco tiempo; prepara el siguiente comando y ejecútalo enseguida.

## 4. Grabar y verificar

Desde la misma carpeta `cmsisdap`, con `.venv` activo y el CH552 todavía en modo de arranque:

```bash
python tools/chprog.py build/main.bin
```

El programa debe detectar el CH552, borrar su firmware anterior, escribir `main.bin`, verificarlo y terminar con `DONE.`. **Este comando programa el CH552; no se usa `pyocd flash` para este paso.**

Si aparece `Could not access USB Device` o un error de permisos, crea la regla para el **modo de arranque**:

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="4348", ATTR{idProduct}=="55e0", MODE="0666"' | sudo tee /etc/udev/rules.d/99-ch55x.rules
sudo udevadm control --reload-rules
```

Desconecta el CH552, vuelve a entrar en modo **BOOT** como en el paso 3 y repite el comando de grabación. Usa el Python del entorno virtual que contiene `pyusb`.

## 5. Comprobar CMSIS-DAP

Desconecta y reconecta el CH552 **sin presionar BOOT**. Ejecuta:

```bash
lsusb -d 1a86:8011
pyocd list
```

El primer comando debe mostrar el CH552 con el firmware CMSIS-DAP; el segundo debe mostrarlo como sonda disponible. `lsusb` confirma que el USB reconoce el dispositivo, mientras que `pyocd list` comprueba que PyOCD puede utilizarlo.

Si `lsusb` muestra `1a86:8011` pero PyOCD informa falta de permisos, crea una regla para el **modo CMSIS-DAP**:

```bash
echo 'SUBSYSTEMS=="usb", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="8011", MODE:="0666"' | sudo tee /etc/udev/rules.d/60-multiprotocol-cmsis-dap.rules
sudo udevadm control --reload-rules
```

Vuelve a conectar el CH552 sin presionar **BOOT** y repite `pyocd list`. Cuando aparezca la sonda, sigue la [guía de PyOCD y conexión SWD](PYOCD.md) para unirla a la placa PY32F003.

## Si algo falla

| Lo que ves | Qué revisar |
| --- | --- |
| `make: sdcc: No such file or directory` | Instala las [herramientas del sistema](../installation.md#1-instalar-herramientas-del-sistema) y vuelve a compilar |
| `ModuleNotFoundError: No module named 'usb'` | Activa `.venv` e instala `pyusb` según la [guía de instalación](../installation.md#3-preparar-python-y-pyocd) |
| `No CH55x device found` | Entra en modo **BOOT**, comprueba `4348:55e0` y ejecuta pronto el comando de grabación |
| `Could not access USB Device` | Aplica la regla udev de `4348:55e0`, reconecta en modo **BOOT** y repite |
| `lsusb` muestra `1a86:8011`, pero `pyocd list` no muestra la sonda | Revisa los permisos del modo CMSIS-DAP, reconecta y repite `pyocd list` |

El identificador `4348:55e0` corresponde al **cargador del CH552**. El identificador `1a86:8011` corresponde al **firmware CMSIS-DAP ya ejecutándose**. Cada modo usa una regla de permisos diferente.
