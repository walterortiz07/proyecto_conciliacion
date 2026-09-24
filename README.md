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

### Todo junto (recomendado)

```
cp .env.example .env      # opcional: todas las variables tienen valor por defecto
docker compose up -d --build
```

Levanta PostgreSQL con las tres bases y los tres servicios en una red
interna. El primer arranque crea las bases; despues cada servicio aplica su
migracion y crea su usuario administrador por su cuenta.

En **Coolify** alcanza con un unico recurso de tipo *Docker Compose* desde
este repositorio. Los dominios se asignan definiendo `SERVICE_FQDN_PYME_8001`,
`SERVICE_FQDN_BANCO_8002` y `SERVICE_FQDN_CONCILIADOR_8000` con la URL de cada
uno; si no se definen, Coolify genera uno por servicio. Para entrar desde el
navegador en una instalacion local, descomentar la linea `ports:` del servicio
correspondiente en el compose.

### Un servicio suelto

Cada carpeta trae su Dockerfile, su `.env.example` y su `docker-compose.yml`
para levantarlo por separado y conectarlo a otra base.

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
