# Kubernetes Rollout and Rollback

## Objective

This project demonstrates how Kubernetes Deployments manage application updates through Rollouts and how to recover from a bad deployment using Rollback.

---

## Environment

- Kubernetes: K3s
- Namespace: rollout-lab
- Workload: Deployment
- Application: Nginx

---

## Initial Deployment

Deployment created with:

```yaml
image: nginx:1.29

---- Apply ----
kubectl apply -f deployment.yaml

----  Verify ----
kubectl rollout status deployment/nginx-app -n rollout-lab

## Revision History

Check deployment revisions:

kubectl rollout history deployment/nginx-app -n rollout-lab

## Application Update

Deployment image updated:
image: nginx:1.30

---- Apply changes ----
kubectl apply -f deployment.yaml

----  Verify rollout ----
kubectl rollout status deployment/nginx-app -n rollout-lab

----  Check revision history ----
kubectl rollout history deployment/nginx-app -n rollout-lab


## ReplicaSet Behavior

Every Deployment revision creates a new ReplicaSet.

Check ReplicaSets:
kubectl get rs -n rollout-lab


## Rollback

Rollback to previous revision:
kubectl rollout undo deployment/nginx-app -n rollout-lab

---- Verify ----
kubectl rollout status deployment/nginx-app -n rollout-lab


## Post-Rollback Verification

Check ReplicaSets:
kubectl get rs -n rollout-lab

---- Result ----

Previous ReplicaSet -> Active
Current ReplicaSet -> Scaled Down
 

## Rollback to Specific Revision

kubectl rollout undo deployment/nginx-app \
--to-revision=1 \
-n rollout-lab


## Useful Commands

* Deployment Status : kubectl rollout status deployment/nginx-app -n rollout-lab

* Revision History : kubectl rollout history deployment/nginx-app -n rollout-lab

* Rollback : kubectl rollout undo deployment/nginx-app -n rollout-lab

* Rollback to Specific Revision : kubectl rollout undo deployment/nginx-app \
                                  --to-revision=1 \
                                  -n rollout-lab

* ReplicaSets: kubectl get rs -n rollout-lab

* Pods : kubectl get pods -n rollout-lab

## Key Learnings
. Deployments maintain revision history.
. Every rollout creates a new ReplicaSet.
. Previous ReplicaSets are retained for rollback purposes.
. Rollback restores the previous stable version.
. Kubernetes performs rolling updates with minimal downtime.
. Rollout and Rollback are essential deployment recovery mechanisms.

## Summary

This project demonstrates:

1.Creating a Deployment.
2.Performing a Rolling Update.
3.Viewing Revision History.
4.Understanding ReplicaSet versioning.
5.Rolling back to a previous stable release.

These are core Kubernetes deployment management skills frequently used by DevOps Engineers, Platform Engineers, and SRE teams.
