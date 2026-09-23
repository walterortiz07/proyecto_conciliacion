# Plataforma de conciliacion con integridad verificable

Tres servicios independientes que trabajan en conjunto: la PYME emite sus
ventas, el banco emite su extracto y el conciliador recibe ambos archivos,
verifica los sellos y concilia.

Cada servicio es una aplicacion FastAPI con su propia base PostgreSQL, su
propio ingreso con sesion, su auditoria y una copia de la cadena de bloques.
Los bloques se propagan entre servicios por una API interna protegida con
un token compartido.

| Carpeta | Puerto | Rol |
|---|---|---|
| `servicio-pyme` | 8001 | Ventas de la empresa y cierre del dia (CSV + sello) |
| `servicio-banco` | 8002 | Movimientos de la cuenta y extracto (CSV o XML + sello) |
| `servicio-conciliador` | 8000 | Recepcion de archivos, verificacion de sellos y conciliacion |

## Puesta en marcha

1. PostgreSQL con tres bases, una por servicio.
2. Copiar el `.env.example` de cada servicio a `.env` y completarlo: cadena
   de conexion, clave de sesion, primer usuario y el mismo `TOKEN_SERVICIO`
   en los tres.
3. Instalar dependencias (`pip install -r requirements.txt`) y arrancar en
   orden: primero `servicio-pyme` y `servicio-banco`, despues
   `servicio-conciliador` (al iniciar sincroniza los bloques que le falten).

Cada servicio aplica su migracion y crea su primer usuario al arrancar
(`python -m app.inicio`). En un solo servidor tambien se pueden levantar los
tres con `docker-compose` desde la carpeta de cada uno.

## Pruebas

No necesitan base de datos:

```
python -m tests.test_sellos          # calculos de la cadena (los tres)
python -m tests.test_conciliacion    # motor de conciliacion (conciliador)
python -m tests.test_arranque        # pantallas y API interna (los tres)
```

## Archivos de ejemplo

Cada servicio trae una carpeta `ejemplos/` con archivos reales para probar el
circuito completo: emitir el extracto en un servicio, cerrar el lote en el
otro y subir ambos al conciliador. Incluye un archivo modificado fuera del
sistema para mostrar la deteccion de alteraciones.

## Notas

* El archivo `.env` y la carpeta `datos/` (archivos emitidos) no se
  versionan.
* Los archivos se sellan con SHA-256 y la huella queda en la cadena: el
  sistema no cifra los datos, garantiza que no cambien sin que se note.
