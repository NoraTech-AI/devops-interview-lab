# Kubernetes CrashLoopBackOff Troubleshooting

This project demonstrates how to intentionally create, debug, and resolve a CrashLoopBackOff scenario in Kubernetes.

---

## 🎯 Objectives

* Understand CrashLoopBackOff state
* Identify root causes of container failure
* Use Kubernetes debugging tools effectively
* Fix application-level misconfiguration
* Observe rolling update behavior

---

## 🧪 Environment

* Kubernetes: K3s
* Namespace: `crash-lab`
* Image: `busybox`
* Workload Type: Deployment

---

## ⚠️ What is CrashLoopBackOff?

CrashLoopBackOff occurs when a container:

1. Starts successfully
2. Exits unexpectedly (non-zero exit code)
3. Kubernetes continuously restarts it
4. Kubernetes applies exponential backoff retries

---

## 🏗️ Initial Faulty Deployment

The application was intentionally misconfigured:

```yaml
containers:
- name: crash-app
  image: busybox
  command: ["sleep", "3600"]
  args:
    - echo "Starting app...; sleep 5; exit 1"
```

---

## 💥 Observed Behavior

Pods entered the following state:

```text
CrashLoopBackOff
Error
Restarting repeatedly
```

---

## 🔍 Debugging Process

### 1. Check Pod Status

```bash
kubectl get pods -n crash-lab
```

---

### 2. Inspect Pod Details

```bash
kubectl describe pod <pod-name> -n crash-lab
```

Key section:

* Events
* Last State
* Exit Code

---

### 3. Check Logs

```bash
kubectl logs <pod-name> -n crash-lab
kubectl logs <pod-name> -n crash-lab --previous
```

Expected output:

```text
Starting app...
(exit 1)
```

---

## 🧠 Root Cause Analysis

The container failed due to:

* Incorrect command/args configuration
* Execution of a script that exits with code 1
* Misconfigured container lifecycle behavior

---

## 🛠️ Fix Applied

The deployment was corrected by removing the faulty `args` section:

```yaml
containers:
- name: crash-app
  image: busybox
  command: ["sleep", "3600"]
```

---

## 🚀 Rollout Behavior

After applying the fix:

```bash
kubectl apply -f deployment.yaml
kubectl rollout restart deployment crash-app -n crash-lab
```

Kubernetes performed a rolling update:

* New ReplicaSet created
* Old ReplicaSet terminated
* New Pods started successfully

---

## 🔄 Final State

```text
STATUS: Running
READY: 1/1
RESTARTS: 0
```

---

## 📌 Key Learnings

* CrashLoopBackOff is caused by application failure, not Kubernetes itself
* Always check:

  * `kubectl describe pod`
  * `kubectl logs`
  * `kubectl logs --previous`
* RollingUpdate creates temporary coexistence of ReplicaSets
* Command/args misconfiguration is a common root cause

---

## 🧰 Useful Commands

```bash
kubectl get pods -n crash-lab
kubectl describe pod -n crash-lab
kubectl logs -n crash-lab <pod>
kubectl logs -n crash-lab <pod> --previous
kubectl get rs -n crash-lab
kubectl rollout status deployment crash-app -n crash-lab
```

---

## 🧭 Summary

This project demonstrated a full troubleshooting lifecycle:

1. Induce failure
2. Observe CrashLoopBackOff
3. Diagnose using logs and events
4. Fix configuration
5. Observe successful rollout

This is a core Kubernetes debugging skill required for DevOps and SRE roles.
