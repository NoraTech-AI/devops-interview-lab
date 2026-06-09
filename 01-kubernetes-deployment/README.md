# Kubernetes Deployment Lab This project demonstrates a full Kubernetes deployment lifecycle using k3s. Features

Namespace creation

Nginx Deployment with replicas

Service exposure via NodePort

Scaling Pods

Rolling Updates

Rollback to previous revision

Architecture

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

Files

manifests/namespace.yaml – namespace definition

manifests/deployment.yaml – nginx deployment

manifests/service.yaml – NodePort service

Apply resources

kubectl apply -f manifests/

Verify

kubectl get all -n devops-lab

Access Nginx

curl http://127.0.0.1:<NODEPORT>

Scale Deployment

kubectl scale deployment nginx-deployment --replicas=5 -n devops-lab

Rolling Update

kubectl apply -f manifests/deployment.yaml

Rollback

kubectl rollout undo deployment nginx-deployment -n devops-lab
