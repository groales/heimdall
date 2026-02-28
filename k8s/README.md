# Heimdall en MicroK8s (NFS)

Base de despliegue para tu clúster MicroK8s con almacenamiento NFS:

- NFS: `NFS_SERVER_IP:/NFS_EXPORT_BASE`

## Qué despliega

- Heimdall con 1 réplica (recomendado para inicio)
- Persistencia en NFS (`/config`)
- Service + Ingress interno
- PodDisruptionBudget

## Archivos

- `00-namespace.yaml`
- `01-pv-pvc.yaml`
- `02-configmap.yaml`
- `03-deployment.yaml`
- `04-service.yaml`
- `05-ingress.yaml`
- `06-pdb.yaml`

## Requisitos previos

1. MicroK8s con addons:

```bash
microk8s enable dns ingress metrics-server
```

2. Directorio NFS creado y con permisos:

```bash
# En servidor NFS
mkdir -p /NFS_EXPORT_BASE/heimdall/config
```

## Personalización rápida

- Edita host en `05-ingress.yaml`:
  - `heimdall.example.com` -> tu dominio real
- Verifica `ingressClassName` en `05-ingress.yaml`:
  - si usas NGINX suele ser `nginx`
  - si usas Traefik, usa el nombre real de tu clase

## Despliegue

```bash
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-pv-pvc.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-deployment.yaml
kubectl apply -f 04-service.yaml
kubectl apply -f 05-ingress.yaml
kubectl apply -f 06-pdb.yaml
```

## Comprobaciones

```bash
kubectl -n heimdall get pods
kubectl -n heimdall get pvc
kubectl -n heimdall get ingress
kubectl -n heimdall logs deploy/heimdall --tail=100
```

## Recomendaciones

- Mantén Heimdall en `replicas: 1` al principio.
- Cuando valides estabilidad, puedes probar `replicas: 2` con el mismo PVC RWX.
