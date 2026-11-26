# Grabar firmware en el PY32F003

[← Anterior: lectura de memoria](LECTURA.md) · **Ruta guiada: paso 7 de 7** · [Finalizar: volver al inicio →](../README.md)

**Flashear** significa escribir en la memoria del PY32F003 un firmware que [ya compilaste](COMPILACION.md). Aquí usarás **PyOCD** y el CH552 en modo CMSIS-DAP para cargar Blink o UART en una placa de práctica. La [guía de PyOCD](PYOCD.md#qué-es-pyocd) explica qué hace cada pieza.

> [!IMPORTANT]
> Estos comandos **reemplazan el firmware existente** en la placa. Si vas a examinar el programa que ya contiene, comienza por la [guía de lectura](LECTURA.md) y sigue las indicaciones del taller antes de grabar un ejemplo.

## Antes de grabar

- [ ] Existe el `.hex` del ejemplo: `build/blink/blink.hex` o `build/uart/uart.hex`.
- [ ] `pyocd list` muestra el CH552 como sonda.
- [ ] `SWCLK`, `SWDIO`, `GND` y la alimentación de 3.3 V están [conectados y revisados](PYOCD.md#conectar-ch552-y-py32f003).

Trabaja desde la raíz del SDK PY32, con el entorno virtual activo:

```bash
cd ~/bugcon2026/py32f003_lib
source ~/bugcon2026/.venv/bin/activate
pyocd list
```

> [!IMPORTANT]
> El archivo de esta página se graba en el **PY32F003**. `unit_ch55x_sdk/.../build/main.bin` es firmware para el **CH552** y se carga como indica la [guía de CMSIS-DAP](CH552_CMSIS_DAP.md).

## Opción principal: PyOCD directo

Elige **solo el ejemplo que compilaste**. El archivo `.hex` ya contiene las direcciones de memoria; `pyocd.yaml` indica dónde está el CMSIS-Pack local.

### Si compilaste Blink

```bash
pyocd flash -t py32f003x4 --config build/blink/pyocd.yaml build/blink/blink.hex
pyocd reset -t py32f003x4 --config build/blink/pyocd.yaml
```

### Si compilaste UART

```bash
pyocd flash -t py32f003x4 --config build/uart/pyocd.yaml build/uart/uart.hex
pyocd reset -t py32f003x4 --config build/uart/pyocd.yaml
```

La primera línea escribe el firmware; la segunda reinicia el PY32 para ejecutarlo. **No ejecutes las dos variantes seguidas**, porque la última reemplazaría el programa anterior.

## Atajo: objetivo de CMake

El proyecto define objetivos que llaman a PyOCD por ti. Para un directorio configurado con `-DAPP=blink` o `-DAPP=uart`, elige la carpeta correspondiente:

```bash
# Si configuraste build/blink con APP=blink
cmake --build build/blink --target flash-reset -j

# Si configuraste build/uart con APP=uart
cmake --build build/uart --target flash-reset -j
```

`--target flash-reset` **compila si hace falta**, llama a `pyocd flash` con el `.hex` y después a `pyocd reset`. Por eso el comando empieza con `cmake --build`, aunque su objetivo final sea grabar. `--target flash` graba sin reiniciar. Escribe `--build` tal cual: `--hash` es un error de escritura.

Si configuraste `-DAPP=all` en `build/all`, los objetivos llevan el nombre del ejemplo:

```bash
cmake --build build/all --target flash_reset_blink -j
cmake --build build/all --target flash_reset_uart -j
```

También aquí debes elegir **uno**; el segundo comando sustituiría Blink por UART.

<details>
<summary>Opción avanzada: grabar un archivo .bin</summary>

Un `.bin` no guarda direcciones de memoria. Para Blink, indica explícitamente el inicio de la flash (`0x08000000`) y reinicia después:

```bash
pyocd flash -t py32f003x4 --config build/blink/pyocd.yaml \
  --base-address 0x08000000 build/blink/blink.bin
pyocd reset -t py32f003x4 --config build/blink/pyocd.yaml
```

Para el primer intento, usa el `.hex` de la opción principal.

</details>

## Comprobar el resultado

PyOCD debe terminar sin errores. Con **Blink**, el LED de la tarjeta debe parpadear. Con **UART**, conecta un adaptador USB-UART a PA0 (TX), PA1 (RX) y GND, abre un terminal a **115200 8N1** y comprueba que el ejemplo devuelve los caracteres recibidos.

| Si ocurre esto | Revisa esto |
| --- | --- |
| `build/... is not a directory` | Falta [configurar y compilar](COMPILACION.md#2-elegir-un-ejemplo-y-configurar-cmake) ese ejemplo |
| `cmake --hash ...` da `Unknown argument` | El comando correcto es `cmake --build ...` |
| `pyocd list` no muestra la sonda | Firmware CMSIS-DAP, cable USB y permisos del [CH552](PYOCD.md#comprobar-el-ch552) |
| PyOCD ve la sonda pero no conecta con el PY32 | `SWDIO`, `SWCLK`, `GND`, 3.3 V y selección `py32f003x4` |
| El `.hex` se grabó pero UART no responde | Cableado TX/RX cruzado, GND común y terminal a 115200 8N1 |

Si falla la grabación, conserva la salida completa de PyOCD y el comando exacto para revisarlos en el taller.
