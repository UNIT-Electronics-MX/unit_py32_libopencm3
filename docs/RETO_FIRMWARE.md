# Firmware didáctico para el reto de dump

[← Anterior: PyOCD y conexión SWD](PYOCD.md) · **Ruta guiada: paso 5 de 7** · [Siguiente: leer el firmware →](LECTURA.md)

`reto_dump` es un firmware reproducible para preparar una placa de práctica antes del taller. Expone una consola UART sencilla y deja pistas deliberadas en el binario para recorrer el flujo completo:

**observar → extraer → verificar → buscar cadenas → desensamblar → comprobar por UART**

No representa un sistema de autenticación seguro ni debe reutilizarse en producción.

## Comportamiento de la placa

| Elemento | Configuración |
| --- | --- |
| Microcontrolador | PY32F003x4 |
| LED | PB5, activo en alto |
| UART | USART2, PA0 TX y PA1 RX |
| Terminal | 115200 baudios, 8N1 |
| Consola | `help`, `status`, `led on`, `led off`, `auth <codigo>` |

La consola muestra cadenas reconocibles, pero mantiene codificado el mensaje final. El participante debe relacionar las cadenas con las funciones y seguir el flujo de autenticación en el desensamblado.

## Preparación por el instructor

Compila en una carpeta independiente:

```bash
cd ~/bugcon2026/py32f003_lib
source ~/bugcon2026/.venv/bin/activate

cmake -S . -B build/reto_dump \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchain-arm-none-eabi.cmake \
  -DAPP=reto_dump
cmake --build build/reto_dump -j

arm-none-eabi-size build/reto_dump/reto_dump.elf
sha256sum build/reto_dump/reto_dump.hex \
  build/reto_dump/reto_dump.bin \
  build/reto_dump/reto_dump-full.bin
```

Los artefactos quedan en `build/reto_dump/`. El `.elf` conserva símbolos y sirve al instructor para validar el ejercicio; el `.hex` es el archivo que se graba; `reto_dump.bin` contiene solo los bytes ocupados por el programa; y `reto_dump-full.bin` tiene 16 KiB, con el espacio restante rellenado con `0xff`, para practicar sin hardware con la misma longitud de una extracción completa.

> [!NOTE]
> El código fuente se incluye para que el taller sea reproducible. Si deseas que el análisis sea un reto, prepara y graba las placas con anticipación y entrega a los participantes solo el hardware; publica la fuente después de la actividad.

## Grabar una placa de práctica

Este paso reemplaza el contenido actual. Confirma que ya guardaste cualquier firmware que necesites conservar:

```bash
pyocd flash -t py32f003x4 \
  --config build/reto_dump/pyocd.yaml \
  build/reto_dump/reto_dump.hex
pyocd reset -t py32f003x4 \
  --config build/reto_dump/pyocd.yaml
```

Conecta un adaptador USB-UART a PA0 (TX de la placa), PA1 (RX de la placa) y GND. Abre el terminal:

```bash
picocom -b 115200 /dev/ttyUSB0
```

Al reiniciar debe aparecer `BUGCON 2026 / DUMP-LAB v1.0`. `help`, `status` y los comandos del LED permiten comprobar la consola antes de entregar la placa.

## Actividad para participantes

1. No abras `examples/reto_dump.c` durante el ejercicio.
2. Obtén dos lecturas de 16 KiB siguiendo [Leer, releer y verificar](LECTURA.md#2-extraer-toda-la-flash).
3. Confirma que ambas tienen el mismo SHA-256.
4. Usa `strings` para encontrar el banner, los comandos y los mensajes de autenticación.
5. Desensambla el dump como Cortex-M0+ Thumb con base `0x08000000`.
6. Localiza el flujo que conduce a `AUTH OK`, recupera el código esperado y determina cómo se transforma el mensaje codificado.
7. Comprueba tu hipótesis en la consola UART.

El objetivo didáctico no es reconstruir el C exacto, sino obtener evidencia suficiente para explicar qué entradas acepta el firmware, qué condición abre la ruta protegida y cómo se forma su respuesta.

## Validación del instructor con símbolos

El ELF ayuda a distinguir problemas del reto de errores de conexión:

```bash
arm-none-eabi-nm -n build/reto_dump/reto_dump.elf | \
  grep -E 'handle_command|authenticate|reveal_flag'

arm-none-eabi-objdump -d -S build/reto_dump/reto_dump.elf | less
```

En el dump entregado no estarán esos nombres. Esa diferencia permite explicar por qué extraer firmware no recupera automáticamente el código fuente original.
