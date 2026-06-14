# Kubernetes ConfigMap & Secret Demo

This project demonstrates how to manage configuration and sensitive data in Kubernetes using ConfigMaps and Secrets, and how to inject them into a running container as environment variables.

---

## Objectives

- Create and use ConfigMap for non-sensitive configuration
- Create and use Secret for sensitive data
- Inject both ConfigMap and Secret into a Kubernetes Deployment
- Validate injected values inside a running container using `kubectl exec`

---

## Environment

- Kubernetes (K3s)
- Namespace: `config-lab`
- Application: Nginx container (used as a dummy app)

---

## Architecture Overview
ConfigMap ───┐
├──> Deployment ───> Pod ───> Environment Variables
Secret ───┘

---

## Step 1: Namespace

Created a dedicated namespace to isolate resources:

```bash
kubectl apply -f namespace.yaml

## Step 2: ConfigMap

ConfigMap stores non-sensitive application configuration.

---- File: configmap.yaml ----

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: config-lab

data:
  APP_ENV: "production"
  APP_NAME: "demo-app"

---- Apply ----

kubectl apply -f configmap.yaml


## Step 3: Secret

Secret stores sensitive data in base64 encoded format.

---- File: secret.yaml ----

apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: config-lab

type: Opaque

data:
  DB_USER: YWRtaW4=
  DB_PASSWORD: c2VjcmV0cGFzcw==


---- Apply ----

kubectl apply -f deployment.yaml


## Step 5: Verification

Check running pod:

kubectl apply -f deployment.yaml
kubectl get pods -n config-lab
kubectl exec -it <pod-name> -n config-lab -- env


## Expected Output 

APP_ENV=production
APP_NAME=demo-app
DB_USER=admin
DB_PASSWORD=secretpass


## Key Concepts Learned
Difference between ConfigMap and Secret
Base64 encoding in Kubernetes Secrets
Environment variable injection using env
Secure separation of sensitive vs non-sensitive data
Runtime configuration without rebuilding container images

## Important Notes
Secrets are not encrypted by default, only encoded (base64)
ConfigMap and Secret are both mounted at runtime
Kubernetes does not differentiate them inside the container
Best practice: use external secret managers (Vault, cloud KMS) in production

