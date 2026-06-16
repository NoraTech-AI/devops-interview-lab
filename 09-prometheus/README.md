#  Prometheus Monitoring with Nginx Exporter

## Objective

Build a basic monitoring stack using Prometheus and Nginx Exporter.

This project demonstrates how Prometheus collects metrics from applications through exporters and how Kubernetes resources are configured for monitoring workloads.

---

## Architecture

```text
                    +----------------+
                    |   Prometheus   |
                    |    :9090       |
                    +--------+-------+
                             |
                             | Scrape
                             |
                             v
                    +----------------+
                    | Nginx Exporter |
                    |    :9113       |
                    +--------+-------+
                             |
                             | Read metrics
                             |
                             v
                    +----------------+
                    |     Nginx      |
                    |  /stub_status  |
                    +----------------+
```

---

## Components

### Namespace

Creates an isolated monitoring environment.

```yaml
monitoring-lab
```

---

### Nginx Application

Deploys an Nginx web server inside Kubernetes.

Resources:

- Deployment
- Service

---

### Prometheus

Deploys Prometheus with a ConfigMap-based configuration.

Resources:

- ConfigMap
- Deployment

Configured scrape jobs:

- Prometheus self-monitoring
- Nginx service
- Nginx Exporter

---

### Nginx Exporter

Deploys the Prometheus Nginx Exporter.

Resources:

- Deployment
- Service

Exporter endpoint:

```text
http://nginx-exporter:9113/metrics
```

---

### Nginx Status Endpoint

Nginx Exporter requires access to:

```text
/stub_status
```

A dedicated ConfigMap was created and mounted into the Nginx container.

Example:

```nginx
location /stub_status {
    stub_status;
    access_log off;
}
```

---

## Deployment Order

Apply resources in the following order:

```bash
kubectl apply -f namespace.yaml

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

kubectl apply -f configmap.yaml
kubectl apply -f prometheus.yaml

kubectl apply -f exporter-deployment.yaml
kubectl apply -f exporter-service.yaml

kubectl apply -f nginx-configmap.yaml
kubectl apply -f deployment.yaml
```

---

## Validation

### Verify Pods

```bash
kubectl get pods -n monitoring-lab
```

Expected:

```text
nginx-app
nginx-exporter
prometheus
```

All should be Running.

---

### Verify Stub Status

```bash
kubectl port-forward svc/nginx-service -n monitoring-lab 8080:80
```

```bash
curl http://localhost:8080/stub_status
```

Expected output:

```text
Active connections: 1
server accepts handled requests
...
```

---

### Verify Exporter Metrics

```bash
kubectl port-forward svc/nginx-exporter -n monitoring-lab 9113:9113
```

```bash
curl http://localhost:9113/metrics
```

Expected:

```text
nginx_connections_active
nginx_connections_accepted
...
```

---

### Verify Prometheus

```bash
kubectl port-forward deployment/prometheus -n monitoring-lab 9090:9090
```

Open:

```text
http://localhost:9090
```

Useful PromQL queries:

```promql
up
```

```promql
nginx_connections_active
```

```promql
nginx_connections_accepted
```

```promql
nginx_connections_handled
```

---

## Troubleshooting

### ConfigMap Not Found

Error:

```text
MountVolume.SetUp failed:
configmap "nginx-config" not found
```

Cause:

Nginx Deployment referenced a ConfigMap that had not been created.

Resolution:

```bash
kubectl apply -f nginx-configmap.yaml
```

---

### Prometheus ConfigMap Not Found

Error:

```text
configmap "prometheus-config" not found
```

Resolution:

```bash
kubectl apply -f configmap.yaml
```

---

### Exporter Returns No Metrics

Cause:

`/stub_status` endpoint is not configured in Nginx.

Resolution:

Verify:

```bash
kubectl exec -it deploy/nginx-app -n monitoring-lab -- cat /etc/nginx/conf.d/default.conf
```

---

### Deployment Validation

```bash
kubectl apply --dry-run=server -f deployment.yaml
```

Useful for catching YAML syntax issues before applying resources.

---

## Key Concepts

- Prometheus architecture
- Pull-based monitoring
- Exporters
- Prometheus scrape jobs
- ConfigMaps
- Volume mounts
- Nginx stub_status
- PromQL basics
- Kubernetes Services
- Deployment rollouts

---

## Interview Notes

### Why is Nginx Exporter required?

Prometheus cannot directly collect detailed Nginx metrics.

Nginx Exporter reads Nginx statistics from `/stub_status` and exposes them in Prometheus format.

---

### Why was stub_status required?

The exporter depends on Nginx runtime statistics.

Without `/stub_status`, the exporter has no data source.

---

### What was the root cause of the monitoring failure?

The Nginx ConfigMap responsible for enabling `/stub_status` was missing.

As a result:

```text
Nginx
  ↓
stub_status unavailable
  ↓
Exporter unable to collect metrics
  ↓
Prometheus receives no Nginx metrics
```

After creating and mounting the ConfigMap, metrics became available immediately.

---

## Useful Commands

```bash
kubectl get all -n monitoring-lab

kubectl get configmap -n monitoring-lab

kubectl describe pod <pod-name> -n monitoring-lab

kubectl logs deployment/prometheus -n monitoring-lab

kubectl logs deployment/nginx-exporter -n monitoring-lab

kubectl rollout restart deployment/nginx-app -n monitoring-lab

kubectl apply --dry-run=server -f deployment.yaml
```

---

## Result

Successfully deployed:

- Nginx
- Nginx Exporter
- Prometheus

Validated end-to-end metrics collection flow:

```text
Nginx
   ↓
stub_status
   ↓
Nginx Exporter
   ↓
Prometheus
   ↓
PromQL Queries
```

