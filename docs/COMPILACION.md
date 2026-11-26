# Compilar firmware para el PY32F003

[← Anterior: instalación](../installation.md) · **Ruta guiada: paso 2 de 7** · [Siguiente: preparar el CH552 →](CH552_CMSIS_DAP.md)

Con esta guía convertirás código C en archivos de firmware **en tu laptop**. Podrás usar Blink, UART o el firmware didáctico `reto_dump` para practicar con una placa PY32F003. Durante el [taller de auditoría](../README.md#qué-harás-durante-el-taller) recibirás una placa preparada para el reto. Si después quieres cargar uno de estos ejemplos, sigue la [guía de flasheo](FLASHEO.md).

> [!IMPORTANT]
> Todos los comandos de esta página se ejecutan desde `~/bugcon2026/py32f003_lib`, la carpeta que contiene `CMakeLists.txt`. Usa `cd` como se muestra antes de compilar.

## 1. Compilar libopencm3 una vez

La biblioteca del SDK se compila antes de los ejemplos:

```bash
cd ~/bugcon2026/py32f003_lib
ls CMakeLists.txt libopencm3/Makefile
make -C libopencm3 TARGETS=py32/f0
ls -lh libopencm3/lib/libopencm3_py32f0.a
```

Si el último archivo existe, la biblioteca quedó lista. Puedes volver a ejecutar `make -C` después de cambios en la biblioteca; no hace falta repetirlo por cada ejemplo.

## 2. Elegir un ejemplo y configurar CMake

`-DAPP=` elige el programa. `-B build/<ejemplo>` crea una carpeta independiente para esa elección; así no se mezclan configuraciones de Blink y UART.

### Blink

```bash
cd ~/bugcon2026/py32f003_lib
cmake -S . -B build/blink \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake \
  -DAPP=blink
```

### UART

```bash
cd ~/bugcon2026/py32f003_lib
cmake -S . -B build/uart \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake \
  -DAPP=uart
```

### Reto de extracción y análisis

```bash
cd ~/bugcon2026/py32f003_lib
cmake -S . -B build/reto_dump \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake \
  -DAPP=reto_dump
```

Este ejemplo ofrece una consola UART con pistas preparadas para practicar extracción y desensamblado. Consulta la [guía del firmware del reto](RETO_FIRMWARE.md) antes de grabarlo.

La configuración prepara las instrucciones de compilación y genera `build/<ejemplo>/pyocd.yaml` para el flasheo posterior. Aún no crea el firmware ni toca el microcontrolador.

## 3. Compilar el ejemplo configurado

Ejecuta **solo** el comando que corresponde a la carpeta que configuraste:

```bash
# Blink
cmake --build build/blink -j

# UART
cmake --build build/uart -j

# Reto de dump
cmake --build build/reto_dump -j
```

La opción es `--build`, no `--hash`. Si ves `CMake Error: Unknown argument -j` después de escribir `cmake --hash ... -j`, corrige el nombre de la opción. Si ves `build/uart is not a directory`, ejecuta primero el paso 2 para UART.

## 4. Comprobar los archivos

```bash
# Si compilaste Blink
ls -lh build/blink/blink.elf build/blink/blink.bin build/blink/blink.hex
arm-none-eabi-size build/blink/blink.elf

# Si compilaste UART
ls -lh build/uart/uart.elf build/uart/uart.bin build/uart/uart.hex
arm-none-eabi-size build/uart/uart.elf

# Si compilaste el reto
ls -lh build/reto_dump/reto_dump.elf build/reto_dump/reto_dump.bin \
  build/reto_dump/reto_dump.hex build/reto_dump/reto_dump-full.bin
arm-none-eabi-size build/reto_dump/reto_dump.elf
```

| Archivo | Para qué sirve |
| --- | --- |
| `.elf` | Ejecutable con información útil para depuración |
| `.hex` | Firmware con direcciones de memoria; es el archivo que usaremos para grabar |
| `.bin` | Bytes de firmware sin dirección de carga |
| `.map` | Mapa de memoria generado por el enlazador |
| `pyocd.yaml` | Configuración de PyOCD que CMake genera para ese directorio |

El ejemplo `reto_dump` también genera `reto_dump-full.bin`: una imagen de 16 KiB rellenada con `0xff`, útil para ensayar el análisis sin placa y comparable con el dump completo que produce `savemem`.

Si ves `Built target uart.elf` y los archivos `uart.elf`, `uart.bin` y `uart.hex`, **UART ya está compilado**. Eso no significa que esté grabado en la tarjeta.

### Opcional: compilar todos los ejemplos

```bash
cd ~/bugcon2026/py32f003_lib
cmake -S . -B build/all \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake \
  -DAPP=all
cmake --build build/all -j
```

En ese caso, los archivos quedan directamente en `build/all/`. El SDK también acepta `reto_dump`, `pwm`, `pwm_pa2`, `adc`, `i2c_pwm`, `i2c_on_off`, `i2c_adc`, `rgb_pa0` y `ws2812_pa0`; consulta los [ejemplos del SDK](https://github.com/Cesarbautista10/py32f003_lib/blob/main/docs/EXAMPLES.md).

## Siguiente paso

Si el CH552 aún no funciona como sonda, sigue la [guía para cargar CMSIS-DAP](CH552_CMSIS_DAP.md). Después, revisa [qué es PyOCD y cómo conectar la sonda](PYOCD.md) antes de grabar el `.hex` en el PY32F003.
