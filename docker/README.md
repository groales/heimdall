# Heimdall - Despliegue con Docker Compose

Dashboard de aplicaciones elegante y minimalista para centralizar el acceso a tus servicios self-hosted.

Referencia oficial de instalación: https://docs.linuxserver.io/images/docker-heimdall

## Características

- 🏠 **Panel unificado**: Acceso rápido a todos tus servicios desde una sola página
- 🎨 **Personalizable**: Soporte para iconos, categorías, fondos y temas
- 🔍 **Búsqueda integrada**: Localiza aplicaciones rápidamente
- 📱 **Diseño responsive**: Interfaz usable en escritorio y móvil
- 🔌 **Enhanced Apps**: Widgets y datos en tiempo real para aplicaciones compatibles
- 💾 **Persistencia simple**: Configuración almacenada en `./config`

## Requisitos Previos

- Docker Engine instalado
- Docker Compose instalado
- Puertos `8080` y `8443` disponibles, o adaptados a tu entorno
- Red Docker externa `proxy` creada, ya que el `compose.yaml` de este repositorio está preparado para conectarse a ella

> ⚠️ **Importante**: Si no vas a usar un proxy inverso genérico, elimina o comenta el bloque `networks` del `compose.yaml` antes del primer arranque.

## Archivos de este Directorio

Este directorio contiene:

- `compose.yaml` - Configuración base del contenedor
- `README.md` - Esta guía de despliegue

---

## Despliegue con Docker Compose

### 1. Clonar el repositorio

```bash
git clone https://github.com/groales/heimdall.git
cd heimdall/docker
```

### 2. Revisar la configuración base

El `compose.yaml` incluido define:

- Imagen `lscr.io/linuxserver/heimdall:latest`
- Publicación de puertos `8080:80` y `8443:443`
- Persistencia local en `./config:/config`
- Variables `PUID`, `PGID`, `TZ` y `ALLOW_INTERNAL_REQUESTS`
- Red externa `proxy`

### 3. Crear la red externa

```bash
docker network create proxy
```

Si no necesitas esa red, elimina el bloque `networks` del `compose.yaml` y omite este paso.

### 4. Ajustar variables y puertos si es necesario

Puedes modificar `compose.yaml` antes del arranque para adaptarlo a tu entorno:

```yaml
environment:
  - PUID=1000
  - PGID=1000
  - TZ=Europe/Madrid
  - ALLOW_INTERNAL_REQUESTS=false

ports:
  - 8080:80
  - 8443:443
```

### 5. Desplegar

```bash
docker compose up -d
```

Para seguir el arranque:

```bash
docker compose logs -f heimdall
```

---

## Acceso Inicial

Una vez desplegado, Heimdall estará disponible en:

```text
http://IP-del-servidor:8080
https://IP-del-servidor:8443
```

El acceso por `8443` usa el certificado interno del contenedor, por lo que el navegador puede mostrar una advertencia si no hay un proxy inverso delante.

### Primera Configuración

1. Accede a Heimdall desde el navegador
2. Entra en el modo de edición con el icono de llave
3. Selecciona `Add Application`
4. Define nombre, icono, URL y descripción de cada servicio
5. Guarda los cambios

### Enhanced Apps

Para aplicaciones compatibles como Sonarr, Radarr, qBittorrent o Pi-hole:

1. Habilita la opción `Enhanced`
2. Añade la API Key correspondiente
3. Revisa las opciones avanzadas disponibles para ese conector

---

## Comandos Útiles

### Ver logs

```bash
docker compose logs -f heimdall
```

### Reiniciar servicio

```bash
docker compose restart heimdall
```

### Actualizar contenedor

```bash
docker compose pull
docker compose up -d
```

### Detener y eliminar

```bash
docker compose down
```

---

## Estructura de Persistencia

```text
Bind mount local:
└── ./config
    ├── appdata
    ├── logs
    └── configuración persistente de Heimdall
```

> 💡 **Tip**: Al usar un bind mount local, el backup es más simple porque basta con copiar o comprimir el directorio `config`.

---

## Configuración Avanzada

### Variables Disponibles

| Variable | Descripción | Valor por defecto en este repo |
|----------|-------------|--------------------------------|
| `PUID` | UID del usuario dentro del contenedor | `1000` |
| `PGID` | GID del grupo dentro del contenedor | `1000` |
| `TZ` | Zona horaria | `Europe/Madrid` |
| `ALLOW_INTERNAL_REQUESTS` | Permite consultas internas desde widgets o enhanced apps | `false` |

### Cambiar puertos

Si `8080` o `8443` están ocupados, cambia el bloque de puertos:

```yaml
ports:
  - 9080:80
  - 9443:443
```

Luego reaplica:

```bash
docker compose up -d
```

### Ajustar permisos y zona horaria

Edita estas variables según tu host:

```yaml
environment:
  - PUID=1000
  - PGID=1000
  - TZ=Europe/Madrid
```

En Linux puedes comprobar UID y GID con:

```bash
id
```

---

## Solución de Problemas

### El puerto ya está en uso

Reasigna los puertos publicados en `compose.yaml` y vuelve a levantar el servicio.

### Error por red `proxy` inexistente

```bash
docker network create proxy
docker compose up -d
```

Si no vas a integrar Heimdall con un proxy inverso, elimina el bloque `networks` del `compose.yaml`.

### Problemas de permisos en `./config`

Revisa los valores `PUID` y `PGID`, corrige permisos en el host si hace falta y vuelve a crear el contenedor:

```bash
docker compose down
docker compose up -d
```

### Widgets o enhanced apps sin datos

Verifica si la integración requiere acceso a endpoints internos y evalúa cambiar:

```yaml
- ALLOW_INTERNAL_REQUESTS=true
```

Hazlo solo si entiendes el impacto de seguridad y el origen de las peticiones.

---

## Seguridad

### Recomendaciones

1. Usa HTTPS real si publicas Heimdall fuera de tu red local
2. Limita el acceso administrativo a redes de confianza o VPN
3. Revisa cuidadosamente el uso de `ALLOW_INTERNAL_REQUESTS`
4. Mantén copia de seguridad periódica del directorio `./config`
5. Actualiza la imagen regularmente

---

## Backup y Restauración

### Backup

```bash
docker compose down
tar -czf heimdall-backup-$(date +%Y%m%d).tar.gz config
docker compose up -d
```

### Restauración

```bash
docker compose down
tar -xzf heimdall-backup-YYYYMMDD.tar.gz
docker compose up -d
```

---

## Recursos

- Documentación oficial: https://docs.linuxserver.io/images/docker-heimdall
- Proyecto Heimdall: https://heimdall.site
- Despliegue Kubernetes: [../k8s/README.md](../k8s/README.md)
