# Kubernetes Horizontal Pod Autoscaler (HPA)

This project demonstrates how to configure and test Horizontal Pod Autoscaler (HPA) in Kubernetes using CPU utilization metrics.

---

## Objectives

* Deploy an application in Kubernetes
* Configure CPU requests and limits
* Create a Horizontal Pod Autoscaler
* Monitor CPU metrics using Metrics Server
* Trigger automatic scaling under load
* Observe scale-up and scale-down behavior
* Troubleshoot image pull failures

---

## Environment

* Kubernetes: K3s
* Namespace: `hpa-lab`
* Metrics Server: Enabled
* HPA API Version: `autoscaling/v2`

---

## Architecture

```text
User Requests
      │
      ▼
Deployment
      │
      ▼
Pods
      ▲
      │
Metrics Server
      ▲
      │
Horizontal Pod Autoscaler
```

---

## Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: hpa-lab
```

---

## Deployment

The deployment defines CPU requests and limits, which are required for HPA to calculate utilization.

```yaml
resources:
  requests:
    cpu: 100m
  limits:
    cpu: 200m
```

Without CPU requests, HPA cannot calculate utilization percentages.

---

## Service

A ClusterIP service exposes the application internally within the cluster.

---

## Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache

  minReplicas: 1
  maxReplicas: 5

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## Metrics Verification

Verify Metrics Server functionality:

```bash
kubectl top nodes
```

Example:

```text
NAME   CPU(cores)   CPU(%)
nops   614m         7%
```

---

## Load Generation

A BusyBox pod was used to continuously send requests to the application.

```bash
kubectl run -it --rm load-generator \
  --image=busybox:1.36 \
  -n hpa-lab -- sh
```

Inside the container:

```bash
while true; do
  wget -q -O- http://php-apache
done
```

---

## HPA Scale-Up Result

Before load:

```text
TARGETS: 0%/50%
REPLICAS: 1
```

After generating traffic:

```text
TARGETS: 43%/50%
REPLICAS: 2
```

The HPA automatically increased the number of replicas in response to higher CPU utilization.

---

## Scale-Down Behavior

After stopping the load generator, CPU utilization dropped back to near zero.

Example:

```text
TARGETS: 0%/50%
REPLICAS: 2
```

The replica count does not immediately return to 1.

This is expected behavior.

Kubernetes intentionally waits before scaling down in order to avoid rapid fluctuations (flapping) caused by temporary spikes in traffic.

After the stabilization period expires, HPA gradually reduces the number of replicas.

---

## Troubleshooting: ErrImagePull / ImagePullBackOff

During implementation, the deployment initially used:

```yaml
image: registry.k8s.io/hpa-example
```

Pods failed to start and entered the following states:

```text
ErrImagePull
ImagePullBackOff
```

Investigation:

```bash
kubectl describe pod <pod-name> -n hpa-lab
```

Events:

```text
403 Forbidden
failed to pull image "registry.k8s.io/hpa-example"
```

Root Cause:

* Kubernetes was functioning correctly.
* The container image could not be downloaded from the registry.
* The registry returned HTTP 403 Forbidden.

Resolution:

The deployment image was replaced with:

```yaml
image: nginx:1.29
```

After updating the image and redeploying, pods started successfully.

---

## Useful Commands

Check HPA:

```bash
kubectl get hpa -n hpa-lab
```

Monitor scaling:

```bash
kubectl get hpa -n hpa-lab -w
```

View resource consumption:

```bash
kubectl top pods -n hpa-lab
```

Inspect deployment:

```bash
kubectl get deployment -n hpa-lab
```

Inspect pods:

```bash
kubectl get pods -n hpa-lab
```

---

## Key Concepts Learned

* Horizontal Pod Autoscaler (HPA)
* Metrics Server integration
* CPU-based auto scaling
* Resource requests and limits
* Scale-up and scale-down behavior
* Troubleshooting ErrImagePull
* Troubleshooting ImagePullBackOff
* Monitoring Kubernetes workloads

---

## Project Outcome

Successfully implemented CPU-based auto scaling in Kubernetes.

Observed:

* Metrics collection
* HPA decision making
* Automatic scale-up
* Scale-down stabilization behavior
* Real-world image pull troubleshooting

This project demonstrates practical Kubernetes autoscaling and troubleshooting skills commonly required in DevOps and SRE roles.
