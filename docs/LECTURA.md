# Leer la memoria y guardar una copia

[← Anterior: firmware del reto](RETO_FIRMWARE.md) · **Ruta guiada: paso 6 de 7** · [Siguiente: flasheo de ejemplos →](FLASHEO.md)

**Leer** significa pedirle al chip los datos de su memoria. Si permite el acceso, PyOCD puede guardarlos en un archivo `.bin` en tu laptop. A ese archivo se le llama **dump** o **volcado**. Es una copia de bytes, no el código C con el que se creó el programa.

La dirección de los datos es **chip → sonda CH552 → PyOCD → archivo en tu laptop**. Es la dirección contraria al [flasheo](FLASHEO.md), que escribe un archivo de tu laptop en el chip.

> [!IMPORTANT]
> Lee únicamente dispositivos propios o para los que tengas autorización. No ejecutes `flash`, `load`, `erase` ni `write` antes de obtener y verificar el dump: esos comandos modifican el objetivo.

## Antes de leer

- Usa una placa de práctica **PY32F003x4** cuyo contenido se pueda leer. El ejemplo de esta página corresponde a ese chip; la placa del reto puede requerir otros valores.
- Conecta el CH552 y la placa como se explica en [PyOCD y programador](PYOCD.md#conectar-ch552-y-py32f003).
- Comprueba que `pyocd list` muestra la sonda. Esto confirma la conexión USB con el CH552; la lectura confirmará si también puedes acceder al chip.
- No ejecutes `pyocd flash` ni los objetivos `flash-reset` antes de guardar la copia: escribirían otro programa en la placa.

Todos los comandos siguientes parten de la raíz del repositorio `~/bugcon2026`. Así, la ruta del pack y los archivos generados son iguales en toda esta guía:

```bash
cd ~/bugcon2026
source .venv/bin/activate
ls -lh py32f003_lib/pyocd-packs/Puya.PY32.pack
```

Si `ls` muestra el archivo, `--pack py32f003_lib/pyocd-packs/Puya.PY32.pack` es la ruta correcta desde `~/bugcon2026`.

## Qué hace `pyocd commander`

`pyocd commander` abre una sesión de depuración y acepta órdenes de lectura, control del procesador y diagnóstico. En este laboratorio usaremos solo estas órdenes no destructivas:

| Orden | Resultado |
| --- | --- |
| `status` | Muestra si el núcleo está ejecutándose o detenido |
| `show map` | Muestra las regiones de memoria que declara el CMSIS-Pack |
| `halt` | Detiene temporalmente el núcleo para tomar una lectura estable |
| `read32 DIRECCIÓN LONGITUD` | Presenta palabras de 32 bits; la longitud se expresa en **bytes** |
| `savemem DIRECCIÓN LONGITUD ARCHIVO` | Copia una región del chip a un archivo local |
| `compare DIRECCIÓN LONGITUD ARCHIVO` | Vuelve a leer y compara el chip contra un archivo, sin escribir |
| `reset` | Reinicia el objetivo; no borra ni programa la flash |
| `exit` | Cierra la sesión interactiva |

`--pack` carga la descripción del fabricante y `-t` selecciona el modelo exacto. `--connect halt` detiene el núcleo al conectar para que la memoria no cambie mientras se inspecciona.

## 1. Reconocer la memoria sin crear un archivo

Abre primero una sesión interactiva:

```bash
pyocd commander -t py32f003x4 \
  --pack py32f003_lib/pyocd-packs/Puya.PY32.pack --connect halt
```

En el prompt `pyocd>` ejecuta una orden por línea:

```text
status
show map
read32 0x08000000 0x20
```

Para el `PY32F003x4`, el mapa debe incluir flash desde `0x08000000` con longitud `0x4000` (16 KiB). Los primeros 8 bytes son normalmente la tabla de vectores:

- La primera palabra suele apuntar a RAM (`0x2000....`) y representa el valor inicial de la pila.
- La segunda suele apuntar a flash (`0x0800....`) y representa el manejador de reinicio. Su bit menos significativo suele ser 1 porque el Cortex-M ejecuta instrucciones Thumb.

Una salida compuesta solo por `0xffffffff` puede indicar flash vacía; un error de acceso puede indicar cableado incorrecto, objetivo equivocado o protección de lectura.

## 2. Extraer toda la flash

Sin cerrar la sesión interactiva, guarda la primera lectura:

```text
savemem 0x08000000 0x4000 firmware-py32-lectura1.bin
```

Después reinicia la placa y sal:

```text
reset
exit
```

También puedes realizar la misma extracción sin entrar al prompt. Cada cadena después de `-c` es una orden de Commander:

```bash
pyocd commander -t py32f003x4 \
  --pack py32f003_lib/pyocd-packs/Puya.PY32.pack --connect halt \
  -c "status" \
     "read32 0x08000000 0x20" \
     "savemem 0x08000000 0x4000 firmware-py32-lectura1.bin" \
     "reset"
```

La dirección `0x08000000` es el inicio y `0x4000` es la **cantidad** de bytes, no la dirección final. El archivo se crea directamente en `~/bugcon2026/`.

## 3. Comprobar el archivo

```bash
wc -c firmware-py32-lectura1.bin
file firmware-py32-lectura1.bin
sha256sum firmware-py32-lectura1.bin
od -Ax -tx1z -N64 firmware-py32-lectura1.bin
```

El tamaño esperado es **16384 bytes**. El tamaño por sí solo no confirma que haya un programa útil: una memoria vacía también puede producir un archivo de ese tamaño. Guarda el hash SHA-256 junto con el dump para poder identificar exactamente la copia analizada.

## 4. Releer y verificar

Haz una segunda extracción y compara la memoria actual contra la primera. Ninguno de estos comandos escribe en la flash:

```bash
pyocd commander -t py32f003x4 \
  --pack py32f003_lib/pyocd-packs/Puya.PY32.pack --connect halt \
  -c "savemem 0x08000000 0x4000 firmware-py32-lectura2.bin" \
     "compare 0x08000000 0x4000 firmware-py32-lectura1.bin" \
     "reset"

sha256sum firmware-py32-lectura1.bin firmware-py32-lectura2.bin
cmp firmware-py32-lectura1.bin firmware-py32-lectura2.bin
```

La verificación correcta presenta `All 16384 bytes match.`, dos hashes iguales y ninguna salida de `cmp`. Si difieren, localiza el primer cambio sin modificar los archivos:

```bash
cmp -l firmware-py32-lectura1.bin firmware-py32-lectura2.bin | head
```

En flash de programa las dos copias deberían ser idénticas. Si no lo son, repite la lectura con el núcleo detenido y revisa alimentación, SWDIO, SWCLK y GND antes de asumir que el firmware cambia.

## 5. Primer análisis del dump

Busca texto con su desplazamiento dentro del archivo:

```bash
strings -a -t x -n 4 firmware-py32-lectura1.bin | head -40
```

Desensambla el binario como ARM Cortex-M/Thumb y conserva las direcciones reales de flash:

```bash
arm-none-eabi-objdump -D -b binary -m arm -M force-thumb \
  --adjust-vma=0x08000000 firmware-py32-lectura1.bin \
  > firmware-py32-lectura1.asm
less firmware-py32-lectura1.asm
```

Un `.bin` no contiene nombres de funciones, tipos ni comentarios. El desensamblador interpreta bytes como instrucciones y puede incluir datos que no son código. Empieza por las cadenas, la tabla de vectores y las referencias a direcciones; después confirma cada hipótesis con el comportamiento observable de la placa.

Para una práctica preparada específicamente para este flujo, consulta el [firmware del reto de dump](RETO_FIRMWARE.md).

## Para otra placa

Usa el modelo de chip, la dirección y la longitud que indique su hoja de datos o el mapa de memoria de su pack. **No copies los valores del PY32F003x4 a otro chip**: podrías leer una zona incorrecta o la lectura fallaría. Primero confirma la autorización, el modelo, el mapa y el acceso; después guarda la copia.
