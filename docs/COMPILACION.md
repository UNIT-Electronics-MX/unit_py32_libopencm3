# Compilar firmware para el PY32F003

[← Instalación](../installation.md) · **Paso 2 de 4** · [Siguiente: PyOCD y programador →](PYOCD.md)

**Compilar** transforma el código C en archivos de firmware **en la laptop**. Este paso no necesita el CH552 ni la tarjeta PY32F003. La grabación del chip está en la [guía de flasheo](FLASHEO.md).

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

La configuración prepara las instrucciones de compilación y genera `build/<ejemplo>/pyocd.yaml` para el flasheo posterior. Aún no crea el firmware ni toca el microcontrolador.

## 3. Compilar el ejemplo configurado

Ejecuta **solo** el comando que corresponde a la carpeta que configuraste:

```bash
# Blink
cmake --build build/blink -j

# UART
cmake --build build/uart -j
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
```

| Archivo | Para qué sirve |
| --- | --- |
| `.elf` | Ejecutable con información útil para depuración |
| `.hex` | Firmware con direcciones de memoria; es el archivo que usaremos para grabar |
| `.bin` | Bytes de firmware sin dirección de carga |
| `.map` | Mapa de memoria generado por el enlazador |
| `pyocd.yaml` | Configuración de PyOCD que CMake genera para ese directorio |

Si ves `Built target uart.elf` y los archivos `uart.elf`, `uart.bin` y `uart.hex`, **UART ya está compilado**. Eso no significa que esté grabado en la tarjeta.

### Opcional: compilar todos los ejemplos

```bash
cd ~/bugcon2026/py32f003_lib
cmake -S . -B build/all \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake \
  -DAPP=all
cmake --build build/all -j
```

En ese caso, los archivos quedan directamente en `build/all/`. El SDK también acepta `pwm`, `pwm_pa2`, `adc`, `i2c_pwm`, `i2c_on_off`, `i2c_adc`, `rgb_pa0` y `ws2812_pa0`; consulta los [ejemplos del SDK](https://github.com/Cesarbautista10/py32f003_lib/blob/main/docs/EXAMPLES.md).

## Siguiente paso

Lee [qué es PyOCD y cómo preparar el CH552](PYOCD.md). La sonda se comprueba y conecta antes de grabar el `.hex` en el PY32F003.
