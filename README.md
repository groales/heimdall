# Heimdall con Docker Compose

## Introducción

Este repositorio despliega [Heimdall](https://heimdall.site), un panel web para organizar enlaces a sitios y aplicaciones. El servicio usa la imagen mantenida por LinuxServer.io y Docker Compose.

La configuración publica Heimdall en el puerto `8080` del equipo y conserva sus datos en `./config`.

## Enlaces de interés

- [Sitio oficial de Heimdall](https://heimdall.site)
- [Documentación de la imagen LinuxServer.io](https://docs.linuxserver.io/images/docker-heimdall/)
- [Imagen `linuxserver/heimdall` en Docker Hub](https://hub.docker.com/r/linuxserver/heimdall)
- [Repositorio de la imagen en GitHub](https://github.com/linuxserver/docker-heimdall)
- [Documentación de Docker Compose](https://docs.docker.com/compose/)

## Requisitos

- Docker Desktop o Docker Engine con Docker Compose.
- Puerto `8080` disponible en el host.

## Despliegue

### Con Git

1. Clona el repositorio y entra en su directorio:

   ```bash
   git clone <URL_DEL_REPOSITORIO>
   cd heimdall
   ```

2. Crea `.env` a partir de la plantilla:

   ```bash
   cp .env.example .env
   ```

3. Ajusta las variables de `.env` si es necesario:

   ```dotenv
   PUID=1000
   PGID=1000
   TZ=Europe/Madrid
   ```

   `PUID` y `PGID` establecen el UID y el GID con los que LinuxServer.io administra los archivos. `TZ` define la zona horaria. Compose también proporciona esos valores por defecto si no están definidos en `.env`.

4. Inicia el servicio:

   ```bash
   docker compose up -d --pull always
   ```

5. Abre [http://localhost:8080](http://localhost:8080).

### Manual, sin Git

1. Crea una carpeta para el proyecto y entra en ella:

   ```bash
   mkdir -p ~/Developer/projects/docker/heimdall
   cd ~/Developer/projects/docker/heimdall
   ```

2. Crea `compose.yaml` con esta configuración:

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
         - "8080:80"
       volumes:
         - ./config:/config
       restart: unless-stopped
   ```

3. (Opcional) Crea `.env` para personalizar los valores:

   ```dotenv
   PUID=1000
   PGID=1000
   TZ=Europe/Madrid
   ```

   Si no creas `.env`, se usarán los valores predeterminados indicados en `compose.yaml`.

4. Inicia el servicio y abre [http://localhost:8080](http://localhost:8080):

   ```bash
   docker compose up -d --pull always
   ```

Docker Compose creará `./config` si aún no existe. Esta carpeta conserva los datos y la configuración de Heimdall.

## Operación

Consultar el estado y los registros:

```bash
docker compose ps
docker compose logs -f heimdall
```

Aplicar cambios o actualizar la imagen:

```bash
docker compose up -d --pull always
```

Detener y eliminar el contenedor y la red del proyecto:

```bash
docker compose down
```

Los datos guardados en `./config` se conservan al detener o recrear el contenedor.

## Puertos y persistencia

| Elemento | Configuración |
| --- | --- |
| Interfaz web | Puerto `8080` del host → puerto `80` del contenedor |
| Configuración y datos | `./config` → `/config` |

## Seguridad y datos locales

`.env` puede contener valores específicos del equipo y está excluido de Git. No añadas contraseñas ni otros secretos a la imagen o al repositorio. La carpeta `config/` también se excluye porque contiene los datos persistentes de la instancia.
