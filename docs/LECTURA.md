# Leer la memoria y guardar una copia

[← PyOCD y programador](PYOCD.md) · [Volver al inicio](../README.md) · [Flasheo de ejemplos →](FLASHEO.md)

**Leer** significa pedirle al chip los datos de su memoria. Si permite el acceso, PyOCD puede guardarlos en un archivo `.bin` en tu laptop. A ese archivo se le llama **dump** o **volcado**. Es una copia de bytes, no el código C con el que se creó el programa.

La dirección de los datos es **chip → sonda CH552 → PyOCD → archivo en tu laptop**. Es la dirección contraria al [flasheo](FLASHEO.md), que escribe un archivo de tu laptop en el chip.

## Antes de leer

- Usa una placa de práctica **PY32F003x4** cuyo contenido se pueda leer. El ejemplo de esta página corresponde a ese chip; la placa del reto puede requerir otros valores.
- Conecta el CH552 y la placa como se explica en [PyOCD y programador](PYOCD.md#conectar-ch552-y-py32f003).
- Comprueba que `pyocd list` muestra la sonda. Esto confirma la conexión USB con el CH552; la lectura confirmará si también puedes acceder al chip.
- No ejecutes `pyocd flash` ni los objetivos `flash-reset` antes de guardar la copia: escribirían otro programa en la placa.

## Ejemplo con PY32F003x4

En el pack incluido en el SDK, la memoria flash de este modelo empieza en `0x08000000` y mide `0x4000` bytes (16 KiB). Desde la carpeta del SDK, ejecuta:

```bash
cd ~/bugcon2026/py32f003_lib
source ~/bugcon2026/.venv/bin/activate
pyocd commander -t py32f003x4 --pack pyocd-packs/Puya.PY32.pack \
  -c "savemem 0x08000000 0x4000 ../firmware-py32.bin"
```

`savemem` lee la dirección inicial y la cantidad de bytes indicadas; después guarda `firmware-py32.bin` en `~/bugcon2026/`. El archivo no se crea durante la compilación: procede de lo que PyOCD pudo leer del chip. Comprueba el tamaño y mira los primeros bytes:

```bash
wc -c ../firmware-py32.bin
od -An -tx1 -N32 ../firmware-py32.bin
```

Si la lectura se completa, el tamaño esperado para este ejemplo es **16384 bytes**. El tamaño por sí solo no confirma que haya un programa útil: una memoria vacía también puede producir un archivo de ese tamaño. Si PyOCD da un error de conexión o de acceso, conserva el mensaje para revisarlo en el taller. Una protección de lectura activa puede impedir obtener la copia.

## Qué puedes observar en la copia

El comando `od` muestra los primeros bytes en hexadecimal. Para buscar fragmentos de texto legible dentro del archivo, prueba:

```bash
strings -n 4 ../firmware-py32.bin | head
```

Si aparecen palabras reconocibles, compáralas con lo que observas al encender la placa o usar sus interfaces. Que no aparezca texto no significa que la lectura haya fallado: muchos programas no guardan mensajes legibles. El archivo binario sirve como punto de partida para el análisis; por sí solo no explica qué hace cada instrucción.

## Para la placa del reto

Usa el modelo de chip, la dirección y la longitud que se indiquen durante la sesión. **No copies los valores del PY32F003x4 a otro chip**: leerías una zona incorrecta o la lectura fallaría. Primero comprueba la conexión y el acceso; después guarda el archivo. La [guía de flasheo](FLASHEO.md) se usa para cargar ejemplos en la placa de práctica cuando se indique.
