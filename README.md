# Infraestructura: Heimdall

# Heimdall Application Dashboard — Despliegue con Docker Compose

Este repositorio proporciona una forma sencilla de desplegar **Heimdall** usando Docker Compose, siguiendo las recomendaciones de LinuxServer.io.

## ¿Qué es Heimdall?

Heimdall es un dashboard elegante y minimalista para organizar todos tus servicios web en un solo lugar. Ideal para:

- 🏠 **Dashboard de servicios** - Organiza accesos a todas tus aplicaciones
- 🎨 **Personalizable** - Temas, iconos y colores configurables
- 🔍 **Búsqueda integrada** - Acceso rápido a servicios
- 📱 **Responsive** - Funciona en móvil, tablet y escritorio
- 🔌 **Enhanced apps** - Widgets con información en tiempo real

## Requisitos

- Docker y Docker Compose instalados en el host
- Puertos 80/443 disponibles (o usar puertos alternativos)
- (Opcional) NPM desplegado para acceso con dominio y SSL

## Qué incluye este stack

- **Servicio**: `heimdall` con la imagen `lscr.io/linuxserver/heimdall:latest`
- **Volúmenes**:
  - `heimdall_config`: Configuración y base de datos
- **Variables de entorno**:
  - `PUID=1000` / `PGID=1000`: Usuario/grupo para permisos de archivos
  - `TZ=Europe/Madrid`: Zona horaria

**⚠️ IMPORTANTE**: El `docker-compose.yml` base **NO publica puertos** por seguridad. Para acceder a Heimdall:
- **Con proxy** (recomendado): Usa archivos override para Traefik o NPM
- **Sin proxy** (acceso directo): Usa `docker-compose.override.standalone.yml.example`

## Despliegue con Docker Compose

### 1. Crear Directorio y Archivos

```bash
# Crear directorio
mkdir heimdall
cd heimdall
```

### 2. Crear docker-compose.yml

Crea el archivo `docker-compose.yml`:

```yaml
services:
  heimdall:
    container_name: heimdall
    image: lscr.io/linuxserver/heimdall:latest
    restart: unless-stopped
    environment:
      PUID: 1000
      PGID: 1000
      TZ: Europe/Madrid
    volumes:
      - heimdall_config:/config

volumes:
  heimdall_config:
    name: heimdall_config
    
# añadir estas líneas al final del archivo para proxy inverso 
networks:
  default:
    external: true
    name: proxy
```

### 3. (Opcional) Configurar Override para Acceso

**⚠️ IMPORTANTE**: El `docker-compose.yml` base **NO publica puertos** por seguridad.

#### Para Traefik

Crea `docker-compose.override.yml`:

```yaml
services:
  heimdall:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.heimdall.rule=Host(`heimdall.tudominio.com`)"
      - "traefik.http.routers.heimdall.entrypoints=websecure"
      - "traefik.http.routers.heimdall.tls.certresolver=letsencrypt"
      - "traefik.http.services.heimdall.loadbalancer.server.port=80"
      - "traefik.http.routers.heimdall.middlewares=security-headers@file"
```

#### Para NPM

No requiere override adicional. Configura el Proxy Host en la UI de NPM apuntando a `heimdall` puerto `80`.

#### Para Acceso Directo (Standalone)

Crea `docker-compose.override.yml`:

```yaml
services:
  heimdall:
    ports:
      - "8080:80"
      - "8443:443"
```

### 4. Desplegar

```bash
# Crear red proxy si no existe
docker network create proxy

# Iniciar servicios
docker compose up -d

# Ver logs
docker compose logs -f heimdall
```

---

## Método Alternativo: Clonar desde Git

Si prefieres usar Git para mantener la configuración actualizada:

```bash
# Clonar repositorio
git clone https://git.ictiberia.com/groales/heimdall.git
cd heimdall

# Para Traefik
cp docker-compose.override.traefik.yml.example docker-compose.override.yml
# Editar override para configurar tu dominio

# Para NPM
cp docker-compose.override.npm.yml.example docker-compose.override.yml

# Para acceso directo
cp docker-compose.override.standalone.yml.example docker-compose.override.yml

# Desplegar
docker network create proxy
docker compose up -d
```

---

## Acceder a la Interfaz

Depende del override usado:

- **Con Traefik**: `https://heimdall.tudominio.com` (ajustar dominio en override)
- **Con NPM**: Configura Proxy Host primero (ver sección NPM), luego `https://heimdall.tudominio.com`
- **Acceso directo**: `http://IP-del-servidor:8080` o `https://IP-del-servidor:8443`

**Nota**: Si no usaste ningún override, el contenedor arranca pero no es accesible (sin puertos publicados).

---

## Configuración Inicial

Al primer acceso, Heimdall muestra una interfaz vacía. Para añadir aplicaciones:

1. Click en **icono de llave** (esquina superior derecha) para editar
2. Click en **Add Application**
3. Rellenar detalles:
   - **Application name**: Nombre del servicio
   - **Colour**: Color del icono
   - **Icon**: Buscar icono o usar URL personalizada
   - **URL**: Dirección completa (ej: `https://jellyfin.tudominio.com`)
   - **Description**: Descripción opcional
4. **Save**

### Enhanced Apps

Heimdall soporta widgets especiales para apps populares:

- **Plex** - Muestra streams activos
- **Sonarr/Radarr** - Próximos estrenos
- **SABnzbd/NZBGet** - Estado de descargas
- **Pihole** - Estadísticas de bloqueo

Para habilitar:
- Marcar **Enable** en la aplicación
- Proporcionar **API Key** del servicio
- Configurar **Enhanced options**

## Integración con Proxy Inverso

Este repositorio incluye tres archivos override:
- `docker-compose.override.traefik.yml.example` - Integración con Traefik
- `docker-compose.override.npm.yml.example` - Integración con NGINX Proxy Manager
- `docker-compose.override.standalone.yml.example` - Acceso directo sin proxy (publica puertos 8080/8443)

**⚠️ Importante**: El `docker-compose.yml` base NO publica puertos. Debes usar uno de estos overrides para acceder a Heimdall.

### Acceso Directo (sin proxy)

**Desde CLI**:
```bash
cp docker-compose.override.standalone.yml.example docker-compose.override.yml
docker compose up -d
```

Acceso:
- **HTTP**: `http://IP-del-servidor:8080`
- **HTTPS**: `https://IP-del-servidor:8443` (certificado autofirmado)

### Con Traefik

**Desde CLI**:
```bash
cp docker-compose.override.traefik.yml.example docker-compose.override.yml
# Editar dominio en docker-compose.override.yml
docker compose up -d
```

Accede a: `https://heimdall.tudominio.com`

### Con NGINX Proxy Manager

#### 1. Desplegar con override

**Desde CLI**:
```bash
cp docker-compose.override.npm.yml.example docker-compose.override.yml
docker compose up -d
```

Esto conecta Heimdall a la red `proxy` compartida con NPM.

#### 2. Configurar Proxy Host en NPM

Accede a NPM (puerto 81) y crea un Proxy Host:

**Pestaña Details**:
- **Domain Names**: `heimdall.tudominio.com`
- **Scheme**: `http`
- **Forward Hostname / IP**: `heimdall` (nombre del contenedor)
- **Forward Port**: `80`
- **Cache Assets**: ✅
- **Block Common Exploits**: ✅
- **Websockets Support**: ❌

**Pestaña SSL**:
- ✅ **Request a new SSL Certificate**
- ✅ **Force SSL**
- ✅ **HTTP/2 Support**
- Email: `tu@email.com`
- ✅ **I Agree to Let's Encrypt ToS**

**Save** y accede a: `https://heimdall.tudominio.com` 🎉

## Despliegue en Kubernetes (MicroK8s)

Si vas a desplegar Heimdall en tu clúster, tienes una base lista en [k8s/README.md](k8s/README.md) con NFS.

## Personalización

### Cambiar tema

1. Click en **icono de llave** → **Settings**
2. **Theme**: Seleccionar tema (Classic, Dark, etc.)
3. **Save**

### Cambiar fondo

1. **Settings** → **Background image**
2. Subir imagen o usar URL
3. **Save**

### Organizar aplicaciones

En modo edición:
- **Drag & drop** para reordenar
- Click en **engranaje** de cada app para editar/eliminar
- Crear **Tags** para agrupar aplicaciones

### Cambiar puertos

**Si usas acceso directo** (override standalone), puedes personalizar los puertos editando `docker-compose.override.standalone.yml.example`:

```yaml
ports:
  - "9080:80"    # HTTP en puerto 9080
  - "9443:443"   # HTTPS en puerto 9443
```

**Si usas proxy** (Traefik o NPM), no necesitas puertos publicados - el acceso es via dominio.

### Variables de entorno

Ajusta PUID/PGID según tu sistema:

```bash
# Ver tu UID/GID
id

# Actualizar en docker-compose.yml
PUID: 1000  # Tu UID
PGID: 1000  # Tu GID
```

## Backup y Restauración

### Backup del volumen

```bash
docker run --rm \
  -v heimdall_config:/config \
  -v $(pwd):/backup \
  alpine tar czf /backup/heimdall-config-$(date +%Y%m%d-%H%M%S).tar.gz -C /config .
```

### Restaurar backup

```bash
# Detener Heimdall
docker compose down

# Restaurar
docker run --rm \
  -v heimdall_config:/config \
  -v $(pwd):/backup \
  alpine tar xzf /backup/heimdall-config-YYYYMMDD-HHMMSS.tar.gz -C /config

# Iniciar Heimdall
docker compose up -d
```

## Solución de problemas

### Puerto ocupado

**Solo aplica si usas acceso directo** (override standalone):

```bash
# Ver qué proceso usa el puerto
Get-NetTCPConnection -LocalPort 8080
# o en Linux
sudo netstat -tulpn | grep :8080

# Cambiar puerto en docker-compose.override.standalone.yml.example
ports:
  - "9080:80"
```

### Contenedor no arranca

```bash
# Ver logs
docker logs heimdall

# Verificar permisos del volumen
docker exec heimdall ls -la /config
```

### Error de permisos

Si ves errores de permisos en logs:

```bash
# Verificar PUID/PGID
id

# Actualizar docker-compose.yml con tus valores
# Recrear contenedor
docker compose down
docker compose up -d
```

### Aplicaciones no se guardan

Verifica que el volumen está montado correctamente:

```bash
docker volume inspect heimdall_config
docker exec heimdall ls -la /config
```

## Actualización

```bash
# Pull nueva imagen
docker compose pull

# Reiniciar con nueva versión
docker compose up -d

# Ver logs
docker logs -f heimdall
```

Heimdall mantiene la configuración entre actualizaciones (volumen persistente).

## Recursos oficiales

- 📘 [Documentación LinuxServer.io](https://docs.linuxserver.io/images/docker-heimdall)
- 🐙 [GitHub - Heimdall](https://github.com/linuxserver/Heimdall)
- 💬 [Comunidad LinuxServer.io](https://discord.gg/YWrKVTn)
- 🐳 [Docker Hub](https://hub.docker.com/r/linuxserver/heimdall)

## Seguridad

### Recomendaciones

1. ✅ **Usar proxy con SSL** (Traefik o NPM) para acceso seguro por dominio
2. ✅ **Evitar acceso directo** en producción - usa override standalone solo para testing
3. ✅ **Restringir acceso** con Access Lists en NPM o middlewares en Traefik
4. ✅ **Backup periódico** del volumen `heimdall_config`
5. ✅ **Actualizar regularmente** con `docker compose pull && docker compose up -d`

### Proteger con Access List (NPM)

En NPM, crea una Access List para restringir acceso:

1. **Access Lists** → **Add Access List**
2. **Name**: `Red local`
3. **Access** → **Allow** → IPs permitidas (ej: `192.168.1.0/24`)
4. **Access** → **Deny** → `0.0.0.0/0`
5. Aplicar al Proxy Host de Heimdall

---

**Versión**: Latest (imagen LinuxServer.io rolling)  
**Última actualización**: Diciembre 2025
