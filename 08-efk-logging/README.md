# 08 - EFK Logging

## Objective

This lab demonstrates centralized logging in Kubernetes using the EFK stack:

- Elasticsearch
- Fluent Bit
- Kibana

The project is divided into two stages:

1. Basic log collection with Fluent Bit
2. Full EFK stack deployment

---

## Architecture

Application Pods generate logs:

Application → stdout/stderr → Fluent Bit → Elasticsearch → Kibana

### Components

| Component | Purpose |
|-----------|----------|
| Log Generator | Produces sample application logs |
| Fluent Bit | Collects logs from Kubernetes nodes |
| Elasticsearch | Stores and indexes logs |
| Kibana | Visualizes logs |
| Kubernetes | Hosts all components |

---

# Part 1: Basic Logging with Fluent Bit

## Namespace

```bash
kubectl create namespace logging-basic
```

## Deploy Log Generator

```bash
kubectl apply -f deployment.yaml
```

## Deploy Fluent Bit

```bash
kubectl apply -f fluent-bit-configmap.yaml
kubectl apply -f fluent-bit-daemonset.yaml
```

---

## Verification

Generated application logs:

```bash
kubectl logs -n logging-basic deploy/log-generator
```

Example:

```text
INFO request processed
ERROR something failed
```

---

Collected logs from Fluent Bit:

```bash
kubectl logs -n logging-basic -l app=fluent-bit-basic
```

Example:

```text
ERROR something failed
```

Result:

- Fluent Bit successfully collected logs
- Logs were read from Kubernetes container log files
- Log forwarding pipeline was verified

---

# Part 2: EFK Stack

## Namespace

```bash
kubectl create namespace logging-efk
```

## Deploy Elasticsearch

```bash
kubectl apply -f elasticsearch.yaml
```

## Deploy Kibana

```bash
kubectl apply -f kibana.yaml
```

## Deploy Fluent Bit

```bash
kubectl apply -f fluent-bit-configmap.yaml
kubectl apply -f fluent-bit-daemonset.yaml
```

---

## Validation Commands

Check Pods:

```bash
kubectl get pods -n logging-efk
```

Check Services:

```bash
kubectl get svc -n logging-efk
```

Check Events:

```bash
kubectl get events -n logging-efk
```

Check Fluent Bit logs:

```bash
kubectl logs -n logging-efk -l app=fluent-bit-efk
```

---

# Troubleshooting

## Issue Encountered

Both Elasticsearch and Kibana remained in:

```text
ImagePullBackOff
```

Observed Pods:

```text
elasticsearch-xxxxx   ImagePullBackOff
kibana-xxxxx          ImagePullBackOff
```

Events:

```text
Back-off pulling image
Error: ImagePullBackOff
```

Affected images:

```text
docker.elastic.co/elasticsearch/elasticsearch:7.17.0
docker.elastic.co/kibana/kibana:7.17.0
```

---

## What ImagePullBackOff Means

Kubernetes attempted to pull a container image but failed repeatedly.

Lifecycle:

```text
Pod Created
    ↓
Pull Image
    ↓
ErrImagePull
    ↓
Retry
    ↓
ImagePullBackOff
```

BackOff means Kubernetes delays retry attempts after repeated failures.

---

## Possible Root Causes

### Invalid Image

```yaml
image: non-existent-image
```

---

### Registry Unreachable

Examples:

```text
i/o timeout
connection refused
```

---

### Registry Access Restrictions

Examples:

```text
403 Forbidden
```

---

### Missing Registry Credentials

Private registries require:

```yaml
imagePullSecrets:
- name: regcred
```

---

### DNS Resolution Issues

Examples:

```text
lookup docker.elastic.co: no such host
```

---

## Troubleshooting Process

Inspect Pod:

```bash
kubectl describe pod <pod-name> -n logging-efk
```

Inspect Events:

```bash
kubectl get events -n logging-efk
```

Test image pull manually:

```bash
sudo crictl pull docker.elastic.co/elasticsearch/elasticsearch:7.17.0
```

Test registry connectivity:

```bash
curl -I https://docker.elastic.co
```

Verify DNS:

```bash
nslookup docker.elastic.co
```

---

## Lab Outcome

### Successfully Completed

- Log generation
- Fluent Bit deployment
- Log collection
- Kubernetes logging workflow
- Logging architecture validation

### Not Completed

- Elasticsearch deployment
- Kibana deployment

Reason:

External image registry access issue resulting in ImagePullBackOff.

---

# Interview Notes

Common Kubernetes logging flow:

```text
Application
    ↓
stdout/stderr
    ↓
Container Logs
    ↓
Fluent Bit / Fluentd
    ↓
Elasticsearch
    ↓
Kibana
```

Common commands used during troubleshooting:

```bash
kubectl describe pod <pod-name>
kubectl get events
kubectl logs <pod-name>
crictl pull <image>
curl -I <registry-url>
```

Important Kubernetes concepts demonstrated:

- Centralized Logging
- Fluent Bit
- DaemonSet
- Container Logs
- Log Aggregation
- EFK Architecture
- ImagePullBackOff Troubleshooting
