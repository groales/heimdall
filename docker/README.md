# Heimdall - Despliegue Docker

Dashboard de aplicaciones elegante y minimalista.

## Características

- 🏠 Organiza todos tus servicios en un solo lugar
- 🎨 Personalizable con temas e iconos
- 🔍 Búsqueda integrada
- 📱 Diseño responsive
- 🔌 Enhanced apps con widgets en tiempo real

## Requisitos

- Docker y Docker Compose
- Puertos 8080/8443 disponibles (o personalizables)
- Red Docker `proxy` creada

## Despliegue rápido

```bash
# Clonar repositorio
git clone https://git.ictiberia.com/groales/heimdall.git
cd heimdall/docker

# Crear red proxy
docker network create proxy

# Desplegar
docker compose up -d
```

## Acceso

- **HTTP**: `http://IP-del-servidor:8080`
- **HTTPS**: `https://IP-del-servidor:8443` (certificado autofirmado)

## Configuración del compose

El `docker-compose.yml` incluye:

- **Imagen**: `lscr.io/linuxserver/heimdall:latest`
- **Puertos**: 8080→80, 8443→443
- **Red**: conectado a red externa `proxy`
- **Volumen**: `heimdall_config` persistente
- **Variables**:
  - `PUID=1000` / `PGID=1000`
  - `TZ=Europe/Madrid`
  - `ALLOW_INTERNAL_REQUESTS=false`

## Integración con Traefik

Para acceso por dominio con SSL automático:

```bash
# Crear .env con tu dominio
echo "DOMAIN_HOST=heimdall.tudominio.com" > .env

# Copiar override
cp docker-compose.override.traefik.yml.example docker-compose.override.yml

# Aplicar
docker compose up -d
```

Acceso: `https://heimdall.tudominio.com`

### Personalizar dominio

Edita `.env`:

```env
DOMAIN_HOST=heimdall.midominio.com
```

O edita directamente `docker-compose.override.yml`:

```yaml
services:
  heimdall:
    labels:
      - traefik.http.routers.heimdall.rule=Host(`heimdall.midominio.com`)
```

## Personalización

### Cambiar puertos

Edita `docker-compose.yml`:

```yaml
ports:
  - "9080:80"
  - "9443:443"
```

Aplica:

```bash
docker compose up -d
```

### Ajustar zona horaria y permisos

Edita variables en `docker-compose.yml`:

```yaml
environment:
  - PUID=1000  # tu UID
  - PGID=1000  # tu GID
  - TZ=Europe/Madrid
```

Obtener tu UID/GID:

```bash
id
```

## Configuración inicial

1. Accede a Heimdall
2. Click en **icono de llave** 🔧
3. **Add Application**:
   - Application name: `Jellyfin`
   - Icon: `jellyfin`
   - URL: `https://jellyfin.tudominio.com`
   - Description: `Servidor multimedia`
4. **Save**

### Enhanced Apps

Para apps compatibles (Sonarr, Radarr, qBittorrent, Pi-hole):

1. Marca **Enable** en la app
2. Añade **API Key**
3. Configura **Enhanced options**

## Operaciones

### Ver logs

```bash
docker compose logs -f heimdall
```

### Reiniciar

```bash
docker compose restart
```

### Detener

```bash
docker compose down
```

### Actualizar

```bash
docker compose pull
docker compose up -d
```

### Backup

```bash
docker run --rm \
  -v heimdall_config:/config \
  -v $(pwd):/backup \
  alpine tar czf /backup/heimdall-backup-$(date +%Y%m%d).tar.gz -C /config .
```

### Restaurar

```bash
docker compose down

docker run --rm \
  -v heimdall_config:/config \
  -v $(pwd):/backup \
  alpine tar xzf /backup/heimdall-backup-YYYYMMDD.tar.gz -C /config

docker compose up -d
```

## Resolución de problemas

### Puerto ocupado

```bash
# Ver qué usa el puerto
netstat -tulpn | grep :8080

# Cambiar puerto en docker-compose.yml
ports:
  - "9080:80"
```

### Error de permisos

Verifica PUID/PGID en compose y reaplica:

```bash
docker compose down
docker compose up -d
```

### Sin red proxy

```bash
docker network create proxy
docker compose up -d
```

## Recursos

- [Documentación oficial](https://docs.linuxserver.io/images/docker-heimdall)
- [GitHub Heimdall](https://github.com/linuxserver/Heimdall)
- [Despliegue Kubernetes](../k8s/README.md)

---

**Repositorio**: [groales/heimdall](https://git.ictiberia.com/groales/heimdall)
