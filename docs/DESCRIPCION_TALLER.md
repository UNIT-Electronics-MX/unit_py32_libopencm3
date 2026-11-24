# Del header al dump: conoce el taller

[Volver al inicio](../README.md)

## Descripción breve

En 90 minutos examinarás una placa embebida preparada para un reto de seguridad. Identificarás interfaces y usarás una sonda CH552 para comprobar si puedes leer el programa guardado en el chip. Si el acceso lo permite, guardarás una copia en tu laptop y conocerás controles de protección para producción.

## Qué harás

Comenzarás con una inspección de la placa para reconocer interfaces y puntos de prueba. Después conectarás el CH552 en modo CMSIS-DAP y comprobarás la comunicación con el dispositivo. Intentarás leer la memoria del chip y guardar los datos en un archivo binario para buscar pistas sobre su funcionamiento. Al final verás una demostración de protección de lectura y conversarás sobre qué controles aplicar antes de llevar un dispositivo a producción.

Ese archivo se llama **dump** o **volcado**: es una copia de los datos que se pudieron leer del chip, no el código fuente original. Trabajarás con las interfaces presentes en el kit y seguirás la secuencia **descubrimiento → acceso → lectura de memoria → análisis**. Puedes consultar la [guía de lectura](LECTURA.md) para ver un ejemplo con una placa PY32F003 de práctica.

## Qué aprenderás

- A reconocer superficies de acceso físico en una placa embebida.
- A establecer comunicación con el objetivo mediante una sonda de depuración.
- A comprobar si la memoria de una placa permite la lectura y a guardar una copia cuando sea posible.
- A relacionar lo que observas en el archivo binario con el comportamiento del dispositivo.
- A identificar controles de acceso y protección adecuados para producción.

## Qué necesitas

Trae una laptop con Linux y un puerto USB disponible. UNIT te prestará durante la sesión la placa objetivo, la sonda CH552 y los cables. El cupo es de 15 personas, según la disponibilidad de kits; devolverás el equipo al terminar.

Puedes [preparar la laptop](../installation.md) antes del evento. La instalación se completa sin las placas.
