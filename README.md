# Del header al dump

### Taller práctico de auditoría de hardware embebido

**BugCON Security Conference 2026** · Villa del Hardware de Electronic Cats<br>
**Jueves 26 de noviembre de 2026 · 10:00** · Instructor: César Bautista · Apoyo: Adrián Rabadán

En este taller examinarás una placa preparada para un reto de seguridad. Identificarás interfaces de acceso y usarás una sonda CH552 para intentar leer el programa guardado en el chip. Si la lectura está permitida, guardarás una copia en tu laptop y explorarás el comportamiento del dispositivo. Al terminar, conocerás controles que ayudan a proteger estas superficies antes de llevar un producto a producción.

## Qué significa «dump»

El **firmware** es el programa que ejecuta el chip. Un **dump** o **volcado** es un archivo con los datos que se pudieron leer de su memoria. Por ejemplo, `firmware-py32.bin` sería una copia de los bytes leídos de una placa PY32F003. Ese archivo no contiene el código C original ni comentarios: contiene datos en formato binario.

| Acción | Dirección de los datos | Resultado |
| --- | --- | --- |
| Compilar | Código C en tu laptop → archivo `.hex` en tu laptop | Preparas un programa para grabarlo |
| Flashear | Archivo `.hex` en tu laptop → memoria del chip | Escribes un programa en la placa |
| Leer o extraer | Memoria del chip → archivo `.bin` en tu laptop | Guardas una copia de lo que se pudo leer |

Si la protección de lectura está activada, la placa puede impedir que obtengas esa copia. Por eso comprobarás el acceso antes de intentar leer y **no flashearás la placa del reto antes de la extracción**: al flashear, reemplazarías su programa.

## Qué harás durante el taller

| Etapa | Tiempo | Tu actividad |
| --- | --- | --- |
| Descubrimiento | 0–15 min | Inspeccionarás la placa e identificarás interfaces y puntos de prueba |
| Acceso | 15–35 min | Conectarás la sonda y comprobarás la comunicación con el dispositivo |
| Extracción | 35–65 min | Intentarás leer la memoria del chip y guardarás una copia si el acceso lo permite |
| Análisis | 65–80 min | Revisarás la copia y relacionarás lo que encuentres con el comportamiento de la placa |
| Cierre | 80–90 min | Verás una demostración de protección de lectura y revisarás controles para producción |

Trabajarás con las interfaces presentes en la placa del reto. Entre las superficies que podrás reconocer se encuentran UART, SPI y SWD/JTAG; las actividades prácticas se centrarán en las que utilice el montaje.

## Antes de asistir

- **Trae una laptop con Linux** (Debian, Ubuntu o Raspberry Pi OS de 64 bits), un puerto USB disponible y conocimientos básicos de terminal y C. Revisa los [requisitos completos](installation.md#requisitos).
- **Prepara tu entorno** con la [guía de instalación](installation.md). Puedes completar ese paso sin las placas.
- **Recibirás el kit en préstamo** durante la sesión: placa objetivo, sonda CH552 y cables. Devuélvelo al terminar.

El cupo es de **15 personas**, por orden de llegada y según la disponibilidad de kits.

## Guías del repositorio

Estas guías forman una ruta de siete pasos para conocer las herramientas y practicar con firmware de ejemplo. Si el CH552 ya aparece en `pyocd list`, puedes omitir su preparación; si la placa del reto ya está preparada por el instructor, puedes pasar de PyOCD directamente a lectura.

| Guía | Cuándo usarla |
| --- | --- |
| [Instalación](installation.md) | Antes del taller, para preparar tu laptop y descargar el código |
| [Compilación](docs/COMPILACION.md) | Cuando quieras generar Blink, UART o el firmware didáctico `reto_dump` |
| [CH552 como CMSIS-DAP](docs/CH552_CMSIS_DAP.md) | Si necesitas grabar el firmware de la sonda antes de usar PyOCD |
| [PyOCD y CH552](docs/PYOCD.md) | Para entender la sonda, verificarla y conectar las señales SWD |
| [Firmware del reto](docs/RETO_FIRMWARE.md) | Para preparar una placa con pistas de análisis y practicar el desensamblado del dump |
| [Lectura de memoria](docs/LECTURA.md) | Para usar `pyocd commander`, extraer dos copias y verificar que sean idénticas |
| [Flasheo](docs/FLASHEO.md) | Para cargar o restaurar un ejemplo en una placa de práctica, cuando se indique |

Los comandos de flasheo sustituyen el firmware existente; durante el reto, sigue las instrucciones del taller antes de usarlos.

## Datos del taller

| | |
| --- | --- |
| **Evento** | [BugCON 2026](https://bugcon.org/), 26 y 27 de noviembre |
| **Lugar** | Villa del Hardware de Electronic Cats |
| **Horario** | Jueves 26 de noviembre, 10:00 |
| **Duración** | 90 minutos |
| **Nivel** | Intermedio |
| **Equipo** | UNIT te presta el kit durante la sesión |
| **Patrocinan** | UNIT Electronics y LATE.AR |

## Contenido del repositorio

| Ruta | Qué encontrarás |
| --- | --- |
| [`installation.md`](installation.md) | Preparación de la laptop y descarga del código |
| [`docs/COMPILACION.md`](docs/COMPILACION.md) | Compilación de ejemplos con CMake |
| [`docs/CH552_CMSIS_DAP.md`](docs/CH552_CMSIS_DAP.md) | Compilación y grabación de CMSIS-DAP en el CH552 |
| [`docs/PYOCD.md`](docs/PYOCD.md) | Uso de PyOCD y conexión de la sonda CH552 |
| [`docs/RETO_FIRMWARE.md`](docs/RETO_FIRMWARE.md) | Preparación y análisis del firmware didáctico `reto_dump` |
| [`docs/LECTURA.md`](docs/LECTURA.md) | Lectura de memoria y guardado de una copia binaria |
| [`docs/FLASHEO.md`](docs/FLASHEO.md) | Grabación de ejemplos en una placa PY32F003 de práctica |
| [`docs/DESCRIPCION_TALLER.md`](docs/DESCRIPCION_TALLER.md) | Descripción ampliada de lo que harás durante el taller |
| [`py32f003_lib/`](https://github.com/Cesarbautista10/py32f003_lib) | SDK PY32F003 y ejemplos |
| `unit_ch55x_sdk/` | [SDK CH55x de UNIT](https://github.com/UNIT-Electronics-MX/unit_ch55x_sdk) para el programador |
