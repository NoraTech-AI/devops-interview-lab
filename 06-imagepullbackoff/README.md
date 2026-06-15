# Kubernetes ImagePullBackOff Troubleshooting

## 🎯 Objective

This project demonstrates how to intentionally create, investigate, and resolve an `ImagePullBackOff` error in Kubernetes.

The goal is to understand how Kubernetes behaves when a container image cannot be downloaded from a container registry.

---

## 🏗️ Environment

* Kubernetes: K3s
* Namespace: `image-lab`
* Workload Type: Deployment
* Container Runtime: containerd

---

## 📚 What is ImagePullBackOff?

`ImagePullBackOff` occurs when Kubernetes cannot download a container image.

The typical sequence is:

```text
Pulling Image
    ↓
ErrImagePull
    ↓
ImagePullBackOff
```

Kubernetes keeps retrying with increasing delays (exponential backoff) until the image becomes available or the configuration is fixed.

---

## 🧪 Faulty Deployment

The deployment was intentionally configured with a non-existent image:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: broken-app
  namespace: image-lab

spec:
  replicas: 1

  selector:
    matchLabels:
      app: broken-app

  template:
    metadata:
      labels:
        app: broken-app

    spec:
      containers:
      - name: broken-app
        image: registry.k8s.io/non-existent-image:latest
```

---

## 💥 Observed Behavior

After deployment:

```bash
kubectl get pods -n image-lab
```

Output:

```text
STATUS: ErrImagePull
STATUS: ImagePullBackOff
```

---

## 🔍 Debugging Process

### 1. Check Pod Status

```bash
kubectl get pods -n image-lab
```

Used to identify failing pods and their current state.

---

### 2. Inspect Pod Events

```bash
kubectl describe pod <pod-name> -n image-lab
```

This command provides detailed information about container startup and image pull failures.

---

### 3. Watch Cluster Events

```bash
kubectl get events -n image-lab --watch
```

Useful for observing image pull attempts and failure messages in real time.

---

## 🚨 Root Cause Analysis

Kubernetes events showed:

```text
Failed to pull image
```

and:

```text
403 Forbidden
```

The image:

```text
registry.k8s.io/non-existent-image:latest
```

does not exist in the registry.

As a result:

1. Image pull failed.
2. Pod entered `ErrImagePull`.
3. Kubernetes retried automatically.
4. Pod entered `ImagePullBackOff`.

---

## 🛠️ Resolution

The image reference was corrected:

```yaml
image: nginx:1.29
```

Apply the updated deployment:

```bash
kubectl apply -f deployment.yaml
```

---

## 🔄 Rollout Behavior

After applying the fix:

* A new ReplicaSet was created.
* The old ReplicaSet was scaled down.
* A new Pod was scheduled.
* The image was downloaded successfully.
* The Pod reached the `Running` state.

---

## ✅ Verification

Check pod status:

```bash
kubectl get pods -n image-lab
```

Expected output:

```text
READY   STATUS
1/1     Running
```

Check ReplicaSets:

```bash
kubectl get rs -n image-lab
```

Expected behavior:

```text
NEW ReplicaSet → 1 replica
OLD ReplicaSet → 0 replicas
```

---

## 🧠 Key Learnings

* `ErrImagePull` is the initial image pull failure.
* `ImagePullBackOff` indicates Kubernetes is retrying with exponential backoff.
* `kubectl describe pod` is the most valuable command for diagnosing image pull issues.
* Kubernetes automatically creates a new ReplicaSet after deployment updates.
* ImagePullBackOff is commonly caused by:

  * Incorrect image names
  * Invalid image tags
  * Private registry authentication issues
  * Registry connectivity problems
  * DNS resolution failures

---

## 🧰 Useful Commands

```bash
kubectl get pods -n image-lab

kubectl describe pod <pod-name> -n image-lab

kubectl get events -n image-lab --watch

kubectl get rs -n image-lab

kubectl rollout status deployment/broken-app -n image-lab
```

---

## 📌 Summary

This project demonstrates a complete troubleshooting workflow for Kubernetes image pull failures:

1. Deploy an invalid image.
2. Observe `ErrImagePull`.
3. Observe `ImagePullBackOff`.
4. Analyze events and pod details.
5. Correct the image reference.
6. Verify successful rollout and pod startup.

This is a common real-world troubleshooting scenario for Kubernetes administrators, DevOps engineers, and SREs.
