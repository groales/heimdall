# Heimdall - Despliegue en Kubernetes

Despliegue de Heimdall sobre MicroK8s con persistencia NFS, Service interno e Ingress TLS.

Referencia oficial de instalación: https://docs.linuxserver.io/images/docker-heimdall

## Características

- ☸️ **Despliegue en Kubernetes**: Recursos separados por manifiesto para facilitar operación y mantenimiento
- 💾 **Persistencia en NFS**: Configuración montada en `/config` mediante PV y PVC
- 🌐 **Exposición por Ingress**: Acceso por dominio con terminación TLS
- ⚙️ **Configuración desacoplada**: Variables principales definidas en `ConfigMap`
- 📦 **Servicio interno**: `ClusterIP` para enrutar tráfico desde el Ingress
- 🩺 **Health checks**: `readinessProbe` y `livenessProbe` incluidos en el Deployment

## Requisitos Previos

- Cluster MicroK8s operativo
- Addons `dns`, `ingress` y `metrics-server` habilitados
- `cert-manager` operativo si quieres emitir certificados automáticamente
- StorageClass `nfs` disponible en el clúster
- Export NFS accesible desde los nodos
- Dominio o subdominio preparado para Heimdall

## Archivos de este Directorio

Este directorio contiene:

- `00-namespace.yaml` - Namespace `heimdall`
- `01-pv-pvc.yaml` - Persistencia NFS con PV y PVC
- `02-configmap.yaml` - Variables `PUID`, `PGID` y `TZ`
- `03-deployment.yaml` - Deployment de Heimdall con probes y recursos
- `04-service.yaml` - Service `ClusterIP` en puerto `80`
- `05-ingress.yaml` - Ingress con TLS y host público
- `README.md` - Esta guía de despliegue

---

## Qué Despliega este Stack

Los manifiestos incluidos crean:

- Un namespace dedicado: `heimdall`
- Un PV y un PVC de `5Gi` con `ReadWriteMany`
- Un `ConfigMap` con `TZ`, `PUID` y `PGID`
- Un `Deployment` con `1` réplica de `lscr.io/linuxserver/heimdall:latest`
- Un `Service` interno tipo `ClusterIP`
- Un `Ingress` con host `heimdall.example.com` y secret TLS `heimdall-tls`

---

## Preparar el Almacenamiento

### 1. Verificar o crear la StorageClass `nfs`

Este despliegue usa la StorageClass `nfs`. Si no existe, puedes crearla con una definición como esta:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs
provisioner: kubernetes.io/nfs
allowVolumeExpansion: true
reclaimPolicy: Retain
```

Después aplícala:

```bash
kubectl apply -f storageclass-nfs.yaml
```

### 2. Preparar el directorio NFS

El PV de este repositorio apunta a:

```text
NFS_SERVER_IP:/NFS_EXPORT_BASE/heimdall/config
```

En el servidor NFS:

```bash
mkdir -p /NFS_EXPORT_BASE/heimdall/config
```

### 3. Ajustar el manifiesto del PV

Antes de desplegar, sustituye los valores placeholder en `01-pv-pvc.yaml`:

- `NFS_SERVER_IP` por la IP o hostname real del servidor NFS
- `/NFS_EXPORT_BASE/heimdall/config` por la ruta exportada real si cambia en tu entorno

---

## Personalización Antes del Despliegue

### Host e Ingress

Edita `05-ingress.yaml` para adaptar:

- `heimdall.example.com` al dominio real
- `ingressClassName: public` si tu controlador usa otra clase
- `cert-manager.io/cluster-issuer: letsencrypt-prod` si prefieres otro issuer

### Configuración del contenedor

Puedes modificar `02-configmap.yaml` para ajustar:

```yaml
data:
  TZ: Europe/Madrid
  PUID: "1000"
  PGID: "1000"
```

### Réplicas y recursos

El `Deployment` parte con una configuración conservadora:

- `replicas: 1`
- `requests`: `100m` CPU y `128Mi` memoria
- `limits`: `500m` CPU y `512Mi` memoria

Si tu entorno lo necesita, ajusta esos valores en `03-deployment.yaml`.

---

## Despliegue en Kubernetes

### 1. Aplicar manifiestos

```bash
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-pv-pvc.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-deployment.yaml
kubectl apply -f 04-service.yaml
kubectl apply -f 05-ingress.yaml
```

También puedes aplicar el directorio completo:

```bash
kubectl apply -f .
```

### 2. Verificar el estado

```bash
kubectl -n heimdall get pods
kubectl -n heimdall get pvc
kubectl -n heimdall get svc
kubectl -n heimdall get ingress
```

### 3. Revisar logs si hace falta

```bash
kubectl -n heimdall logs deploy/heimdall --tail=100
```

---

## Acceso Inicial

Una vez propagado el Ingress y emitido el certificado, accede usando el host configurado en `05-ingress.yaml`.

```text
https://heimdall.example.com
```

### Primera Configuración

1. Accede a Heimdall desde el navegador
2. Entra en el modo de edición con el icono de llave
3. Añade tus aplicaciones con nombre, icono y URL real
4. Guarda los cambios

### Enhanced Apps

Para aplicaciones compatibles, habilita la opción `Enhanced` y añade las credenciales o API keys necesarias desde la propia interfaz.

---

## Comandos Útiles

### Estado general

```bash
kubectl -n heimdall get all
```

### Ver configuración aplicada

```bash
kubectl -n heimdall get configmap heimdall-config -o yaml
kubectl -n heimdall describe ingress heimdall
```

### Reiniciar el despliegue

```bash
kubectl -n heimdall rollout restart deploy/heimdall
```

### Seguir logs

```bash
kubectl -n heimdall logs deploy/heimdall -f
```

---

## Estructura de Persistencia

```text
NFS export:
└── /NFS_EXPORT_BASE/heimdall/config
    └── datos persistentes montados en /config
```

El PVC `heimdall-config` usa `ReadWriteMany`, lo que facilita mover el pod entre nodos o escalar con almacenamiento compartido si tu backend NFS lo soporta correctamente.

---

## Solución de Problemas

### El PVC no enlaza con el PV

Revisa que coincidan:

- `storageClassName: nfs`
- `volumeName: heimdall-config-pv`
- capacidad y modos de acceso compatibles

### El pod no arranca por error de NFS

Verifica la IP del servidor, la ruta exportada y los permisos del recurso compartido. Después revisa eventos:

```bash
kubectl -n heimdall describe pod -l app=heimdall
```

### El Ingress no responde

Confirma que:

- el DNS apunta al controlador de ingreso
- `ingressClassName` coincide con tu instalación
- el issuer de `cert-manager` existe

Comprueba detalles con:

```bash
kubectl -n heimdall describe ingress heimdall
```

### Certificado TLS no emitido

Si estás probando el entorno, puede ser preferible cambiar a `letsencrypt-staging` antes de usar producción.

---

## Seguridad

### Recomendaciones

1. Usa un dominio real y TLS válido para exponer Heimdall
2. Limita el acceso al panel mediante red privada, VPN o controles perimetrales
3. Protege el export NFS y permite acceso solo desde nodos autorizados
4. Mantén copias de seguridad del directorio persistente montado en `/config`
5. Revisa periódicamente la imagen y actualiza el Deployment

---

## Backup y Restauración

### Backup

Al usar NFS, la copia de seguridad debe realizarse sobre el directorio exportado que respalda `/config`.

Ejemplo en el servidor NFS:

```bash
tar -czf heimdall-k8s-backup-$(date +%Y%m%d).tar.gz /NFS_EXPORT_BASE/heimdall/config
```

### Restauración

Restaura el contenido del directorio exportado, verifica permisos y reinicia el Deployment:

```bash
kubectl -n heimdall rollout restart deploy/heimdall
```

---

## Recursos

- Documentación principal: [../README.md](../README.md)
- Despliegue Docker: [../docker/README.md](../docker/README.md)
- Documentación oficial: https://docs.linuxserver.io/images/docker-heimdall
- Proyecto Heimdall: https://heimdall.site
