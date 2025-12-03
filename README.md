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
- **Puertos**:
  - `8080`: HTTP (interfaz web)
  - `8443`: HTTPS (certificado autofirmado)
- **Volúmenes**:
  - `heimdall_config`: Configuración y base de datos
- **Red**: `proxy` (compartida con NPM/Traefik)
- **Variables de entorno**:
  - `PUID=1000` / `PGID=1000`: Usuario/grupo para permisos de archivos
  - `TZ=Europe/Madrid`: Zona horaria

## Pasos de despliegue

### Opción 1: Docker Compose (Línea de comandos)

#### 1. Clonar el repositorio

```bash
git clone https://git.ictiberia.com/groales/heimdall
cd heimdall
```

#### 2. Levantar el stack

```bash
docker compose up -d
```

### Opción 2: Desplegar desde Portainer (Recomendado)

#### 1. Desplegar stack de Heimdall

**Stacks** → **Add stack**
- **Name**: `heimdall`
- **Build method**: 
  - **Git Repository**:
    - Repository URL: `https://git.ictiberia.com/groales/heimdall`
    - Repository reference: `refs/heads/main`
    - Compose path: `docker-compose.yml`
  - O **Web editor**: Pegar contenido de `docker-compose.yml`
- **Deploy the stack**

#### 2. Verificar despliegue

**Stacks** → `heimdall` → Ver logs del contenedor

### 3. Verificar el estado

```bash
docker ps --filter name=heimdall
```

### 4. Acceder a la interfaz

Abre tu navegador en:
- **HTTP**: `http://IP-del-servidor:8080`
- **HTTPS**: `https://IP-del-servidor:8443` (certificado autofirmado)

## Configuración inicial

Al primer acceso, Heimdall muestra una interfaz vacía. Para añadir aplicaciones:

1. Click en **icono de llave** (esquina superior derecha) para editar
2. Click en **Add Application**
3. Rellenar detalles:
   - **Application name**: Nombre del servicio
   - **Colour**: Color del icono
   - **Icon**: Buscar icono o usar URL personalizada
   - **URL**: Dirección completa (ej: `https://portainer.tudominio.com`)
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

## Integración con NGINX Proxy Manager

### 1. Conectar a red proxy

La red `proxy` se crea automáticamente al desplegar el stack (compartida con NPM).

### 2. Configurar Proxy Host en NPM

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

Si los puertos 8080/8443 están ocupados, edita `docker-compose.yml`:

```yaml
ports:
  - "9080:80"    # HTTP en puerto 9080
  - "9443:443"   # HTTPS en puerto 9443
```

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

```bash
# Ver qué proceso usa el puerto
Get-NetTCPConnection -LocalPort 8080
# o en Linux
sudo netstat -tulpn | grep :8080

# Cambiar puerto en docker-compose.yml
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

1. ✅ **Usar NPM con SSL** para acceso seguro por dominio
2. ✅ **Restringir acceso** al puerto 8080 con firewall (si no usas proxy)
3. ✅ **Backup periódico** del volumen `heimdall_config`
4. ✅ **Actualizar regularmente** con `docker compose pull && docker compose up -d`
5. ✅ **Usar contraseñas fuertes** si habilitas autenticación

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
