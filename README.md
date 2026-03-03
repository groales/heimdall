# Heimdall

Repositorio organizado por modo de despliegue.

## Estructura

- `docker/` - Despliegue con Docker Compose (stack principal)
- `k8s/` - Despliegue en Kubernetes (MicroK8s)

## Guías

- Docker Compose: [docker/README.md](docker/README.md)
- Kubernetes: [k8s/README.md](k8s/README.md)

## Uso rápido

### Docker

```bash
cd docker
# opcional: cp docker-compose.override.traefik.yml.example docker-compose.override.yml
docker compose up -d
```

### Kubernetes

```bash
cd k8s
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-pv-pvc.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-deployment.yaml
kubectl apply -f 04-service.yaml
kubectl apply -f 05-ingress.yaml
kubectl apply -f 06-pdb.yaml
```
