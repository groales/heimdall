# Heimdall

Dashboard de aplicaciones elegante y minimalista para centralizar accesos a tus servicios self-hosted en una sola página.

Referencia oficial de instalación: https://docs.linuxserver.io/images/docker-heimdall

## Características

- 🏠 **Panel unificado**: Organiza todos tus servicios en una única portada
- 🎨 **Personalizable**: Soporte para iconos, categorías, temas y enlaces rápidos
- 🔍 **Búsqueda integrada**: Localiza aplicaciones rápidamente desde la interfaz
- 📱 **Responsive**: Experiencia adaptada a escritorio y móvil
- 🔌 **Enhanced Apps**: Widgets y datos en tiempo real para aplicaciones compatibles
- 💾 **Persistencia sencilla**: Configuración almacenada en un directorio dedicado

## Modos de Despliegue Disponibles

Este repositorio incluye dos formas de desplegar Heimdall:

- `docker/` - Despliegue con Docker Compose
- `k8s/` - Despliegue en Kubernetes sobre MicroK8s

## Requisitos Previos

### Para Docker Compose

- Docker Engine instalado
- Docker Compose instalado
- Puertos `8080` y `8443` disponibles, o cambiados en `docker/compose.yaml`
- Red Docker externa `proxy` creada si quieres mantener la integración con proxy inverso genérico

### Para Kubernetes

- Cluster MicroK8s operativo
- Addons `dns`, `ingress` y `metrics-server` habilitados
- StorageClass `nfs` disponible
- Export NFS preparado para la persistencia de Heimdall
- Dominio o hostname definido para el Ingress

## Archivos de este Repositorio

Este repositorio contiene dos guías de despliegue separadas:

- `docker/compose.yaml` - Configuración base para Docker Compose
- `docker/README.md` - Guía detallada de despliegue con Docker
- `k8s/*.yaml` - Manifiestos para Kubernetes
- `k8s/README.md` - Guía detallada de despliegue con MicroK8s
- `README.md` - Esta documentación general

---

## Elegir el Método de Despliegue

### Opción 1. Docker Compose

Recomendado si buscas un despliegue rápido en un único host.

```bash
git clone https://github.com/groales/heimdall.git
cd heimdall/docker

# Crear la red externa requerida por este compose
docker network create proxy

# Iniciar Heimdall
docker compose up -d
```

Guía completa: [docker/README.md](docker/README.md)

### Opción 2. Kubernetes (MicroK8s)

Recomendado si ya trabajas con un clúster y persistencia NFS.

```bash
git clone https://github.com/groales/heimdall.git
cd heimdall/k8s

kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-pv-pvc.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-deployment.yaml
kubectl apply -f 04-service.yaml
kubectl apply -f 05-ingress.yaml
```

Guía completa: [k8s/README.md](k8s/README.md)

---

## Acceso Inicial

### Docker Compose

Tras el despliegue, Heimdall quedará disponible en:

```text
http://IP-del-servidor:8080
https://IP-del-servidor:8443
```

### Kubernetes

Accede mediante el host configurado en `k8s/05-ingress.yaml`.

---

## Primera Configuración

1. Accede a Heimdall desde el navegador
2. Entra en el modo de edición usando el icono de llave
3. Añade tus aplicaciones con nombre, icono y URL real
4. Si una app es compatible, habilita `Enhanced App` y configura su API Key

---

## Comandos Útiles

### Docker Compose

```bash
cd docker

# Ver logs
docker compose logs -f heimdall

# Reiniciar servicio
docker compose restart heimdall

# Actualizar imagen
docker compose pull
docker compose up -d
```

### Kubernetes

```bash
cd k8s

# Estado general
kubectl -n heimdall get pods
kubectl -n heimdall get pvc
kubectl -n heimdall get ingress

# Logs
kubectl -n heimdall logs deploy/heimdall --tail=100
```

---

## Seguridad

- Publica el servicio detrás de HTTPS siempre que sea posible
- Restringe el acceso administrativo a redes de confianza o VPN
- Revisa qué aplicaciones usan `Enhanced Apps` y protege sus API keys
- Haz copia de seguridad periódica del directorio de configuración o del PVC

---

## Recursos

- Documentación Docker: [docker/README.md](docker/README.md)
- Documentación Kubernetes: [k8s/README.md](k8s/README.md)
- Documentación oficial: https://docs.linuxserver.io/images/docker-heimdall
- Proyecto Heimdall: https://heimdall.site
