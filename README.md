# Heimdall con Docker Compose

## Introducción

Este proyecto despliega [Heimdall](https://heimdall.site), un panel web para organizar enlaces a sitios y aplicaciones. Utiliza la imagen de [LinuxServer.io](https://docs.linuxserver.io/images/docker-heimdall/) y persiste la configuración y los datos en `./config`.

El despliegue admite dos modos: publicar los puertos HTTP/HTTPS directamente en el host, o conectar Heimdall a una red Docker externa llamada `proxy` para usarlo detrás de un proxy inverso.

## Enlaces de interés

- [Sitio oficial de Heimdall](https://heimdall.site)
- [Documentación de la imagen LinuxServer.io](https://docs.linuxserver.io/images/docker-heimdall/)
- [Imagen en Docker Hub](https://hub.docker.com/r/linuxserver/heimdall)
- [Repositorio de la imagen en GitHub](https://github.com/linuxserver/docker-heimdall)
- [Documentación de Docker Compose](https://docs.docker.com/compose/)

## Requisitos

- Docker Desktop o Docker Engine con Docker Compose.
- Para acceso directo, puertos `80` y `443` libres en el host.
- Para usar un proxy inverso, una red Docker externa llamada `proxy` y el proxy conectado a esa red.

## Despliegue

### Con Git

```bash
git clone <URL_DEL_REPOSITORIO>
cd heimdall
cp .env.example .env
```

Elige un modo antes de iniciarlo:

- **Acceso directo:** no crees `compose.override.yaml`.
- **Proxy inverso:** copia `compose.override.example.yaml` a `compose.override.yaml` y asegúrate de que la red externa `proxy` existe.

Inicia Heimdall:

```bash
docker compose up -d --pull always
```

### Manual, sin Git

Crea la carpeta y entra en ella:

```bash
mkdir -p ~/Developer/projects/docker/heimdall
cd ~/Developer/projects/docker/heimdall
```

Crea `compose.yaml` con esta configuración:

```yaml
services:
  heimdall:
    image: lscr.io/linuxserver/heimdall:latest
    container_name: heimdall
    environment:
      PUID: ${PUID:-1000}
      PGID: ${PGID:-1000}
      TZ: ${TZ:-Europe/Madrid}
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./config:/config
    restart: unless-stopped
```

Crea `.env` si quieres personalizar las variables. Para el modo de proxy, crea también `compose.override.yaml` con el siguiente contenido y crea la red `proxy` como se explicó anteriormente:

```yaml
services:
  heimdall:
    ports: !reset []
    networks:
      - proxy

networks:
  proxy:
    external: true
    name: proxy
```

Inicia el servicio y selecciona la URL correspondiente al modo configurado:

```bash
docker compose up -d --pull always
```

## Configuración

### Variables de entorno

Copia la plantilla y edita `.env` si hace falta:

```bash
cp .env.example .env
```

```dotenv
PUID=1000
PGID=1000
TZ=Europe/Madrid
```

`PUID` y `PGID` indican el UID y el GID con los que LinuxServer.io gestiona los archivos. `TZ` configura la zona horaria. `compose.yaml` usa esos valores como predeterminados si no se definen en `.env`.

### Acceso directo desde el host

El archivo principal `compose.yaml` publica estos puertos:

| Host | Contenedor | Uso |
| --- | --- | --- |
| `80` | `80` | Interfaz HTTP |
| `443` | `443` | Interfaz HTTPS |

Abre `http://localhost` o `https://localhost`. La imagen puede generar certificados autofirmados; el navegador puede advertirlo al acceder por HTTPS hasta que se configure un certificado confiable.

### Detrás de un proxy inverso

El archivo local `compose.override.yaml` elimina las publicaciones de puertos del archivo principal y conecta Heimdall a la red externa `proxy`. Ese archivo está excluido de Git porque contiene una decisión local del despliegue.

Para configurar este modo en un clon o una instalación manual, copia la plantilla versionada:

```bash
cp compose.override.example.yaml compose.override.yaml
```

Asegúrate de que la red `proxy` ya existe y que tu proxy inverso está conectado a ella. Si no existe, créala:

```bash
docker network create proxy
```

Configura el proxy para llegar al servicio `heimdall` en el puerto interno `80` y gestiona TLS en el proxy. Al activar el override, Heimdall no publica los puertos `80` ni `443` en el host.

## Operación

Consultar el estado y los registros:

```bash
docker compose ps
docker compose logs -f heimdall
```

Actualizar la imagen y recrear el contenedor:

```bash
docker compose up -d --pull always
```

Detener y eliminar el contenedor y la red creada por Compose:

```bash
docker compose down
```

`docker compose down` no elimina los datos almacenados en `./config` ni la red externa `proxy`.

## Persistencia y archivos locales

- `./config` se monta en `/config` dentro del contenedor y contiene los datos persistentes de Heimdall.
- `.env` contiene valores locales y está excluido de Git; `.env.example` es la plantilla versionada.
- `compose.override.yaml` permite personalizar el despliegue local y está excluido de Git; `compose.override.example.yaml` es la plantilla versionada.
- `config/` también está excluido de Git.

No incluyas contraseñas ni otros secretos en el repositorio o en la imagen. Para secretos de producción, utiliza un mecanismo específico de gestión de secretos.
