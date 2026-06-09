# Kubernetes Deployment Lab

A hands-on Kubernetes lab demonstrating:

- Namespace management
- Deployments
- ReplicaSets
- Services
- Scaling
- Rolling Updates
- Rollbacks

Environment:
- Kubernetes: k3s
- OS: Ubuntu
- Container Runtime: containerd


## Features

- Namespace creation
- Nginx Deployment with multiple replicas
- Service exposure using NodePort
- Scaling Pods
- Rolling Updates
- Rollback to previous revision

## Architecture

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods
    │
    ▼
Service (NodePort)
```

## Project Structure

```text
01-kubernetes-deployment/
├── manifests/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── screenshots/
└── README.md
```

## Create Namespace

```bash
kubectl apply -f manifests/namespace.yaml
```

## Deploy Nginx

```bash
kubectl apply -f manifests/deployment.yaml
```

## Create Service

```bash
kubectl apply -f manifests/service.yaml
```

## Verify Resources

```bash
kubectl get all -n devops-lab
```

## Access Application

```bash
curl http://127.0.0.1:<NODEPORT>
```

## Scale Deployment

Scale to 5 replicas:

```bash
kubectl scale deployment nginx-deployment \
  --replicas=5 \
  -n devops-lab
```

## Rolling Update

Update image version in `deployment.yaml` and apply:

```bash
kubectl apply -f manifests/deployment.yaml
```

Check rollout status:

```bash
kubectl rollout status deployment nginx-deployment \
  -n devops-lab
```

## Rollback

View revision history:

```bash
kubectl rollout history deployment nginx-deployment \
  -n devops-lab
```

Rollback to previous version:

```bash
kubectl rollout undo deployment nginx-deployment \
  -n devops-lab
```

## Learned Concepts

- Kubernetes Namespace
- Deployment
- ReplicaSet
- Pod Lifecycle
- Service
- NodePort
- Scaling
- Rolling Update
- Rollback
