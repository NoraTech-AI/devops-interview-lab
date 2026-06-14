# Kubernetes Service and Ingress

This project demonstrates how to expose applications running in Kubernetes using Services and Ingress resources.

## Objectives

* Deploy Nginx applications in Kubernetes
* Expose applications using ClusterIP Services
* Configure Traefik Ingress Controller
* Implement Host-Based Routing
* Implement Path-Based Routing
* Validate routing behavior

---

## Environment

* Kubernetes: K3s
* Ingress Controller: Traefik
* Application: Nginx
* Namespace: ingress-lab

---

## Project Structure

```text
02-service-and-ingress/
├── manifests/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── deployment-app1.yaml
│   ├── deployment-app2.yaml
│   ├── service-app1.yaml
│   ├── service-app2.yaml
│   ├── ingress-host-routing.yaml
│   └── ingress-path-routing.yaml
│
├── screenshots/
│   ├── ingress-list.png
│   ├── host-routing.png
│   ├── path-routing.png
│   └── ingress-describe.png
│
└── README.md
```

---

## Scenario 1 - Basic Ingress

### Architecture

```text
nginx.local
      │
      ▼
Ingress
      │
      ▼
nginx-service
      │
      ▼
nginx Pod
```

### Test

```bash
curl http://nginx.local
```

Expected result:

```text
HTTP 200 OK
```

---

## Scenario 2 - Host-Based Routing

### Architecture

```text
app1.local
      │
      ▼
Ingress
      │
      ▼
nginx-app1-service
      │
      ▼
nginx-app1 Pod


app2.local
      │
      ▼
Ingress
      │
      ▼
nginx-app2-service
      │
      ▼
nginx-app2 Pod
```

### Test

```bash
curl -I http://app1.local
curl -I http://app2.local
```

Expected result:

```text
HTTP/1.1 200 OK
```

---

## Scenario 3 - Path-Based Routing

### Architecture

```text
nginx.local/app1
        │
        ▼
nginx-app1-service

nginx.local/app2
        │
        ▼
nginx-app2-service
```

### Test

```bash
curl -I http://nginx.local/app1
curl -I http://nginx.local/app2
```

### Result

Ingress successfully routes requests to backend services.

The default Nginx image does not provide content for `/app1` and `/app2` paths, therefore Nginx returns:

```text
HTTP 404 Not Found
```

This confirms that routing is functioning correctly and requests are reaching the backend pods.

---

## Verification Commands

### List Resources

```bash
kubectl get all -n ingress-lab
```

### List Ingresses

```bash
kubectl get ingress -n ingress-lab
```

### Describe Ingress

```bash
kubectl describe ingress nginx-ingress -n ingress-lab

kubectl describe ingress host-routing-ingress -n ingress-lab

kubectl describe ingress path-routing-ingress -n ingress-lab
```

---

## Key Concepts Practiced

* Kubernetes Deployment
* Kubernetes Service
* ClusterIP
* Ingress
* Traefik Ingress Controller
* Host-Based Routing
* Path-Based Routing
* Kubernetes Networking
* Traffic Routing
* Service Discovery

---

## Screenshots

The screenshots directory contains:

* Ingress resource list
* Host-based routing validation
* Path-based routing validation
* Ingress descriptions

---

## Learning Outcomes

By completing this project, the following concepts were practiced:

* Exposing applications using Kubernetes Services
* Configuring Traefik as an Ingress Controller
* Routing traffic using host-based rules
* Routing traffic using path-based rules
* Troubleshooting Ingress routing issues
* Validating Kubernetes networking resources

