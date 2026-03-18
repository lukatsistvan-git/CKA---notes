# Rolling Updates and Rollbacks

## Overview

The Kubernetes Deployment resource enables you to update applications and roll back changes without downtime.
Each update creates a new revision, which is tracked automatically.
This allows:
- controlled updates
- version history tracking
- quick rollback to a previous state

---

## Rolling Update (Default Strategy)

The default update strategy in Kubernetes is RollingUpdate.
It replaces old Pods with new ones gradually, ensuring the application remains available.

### How It Works

- A new ReplicaSet is created for the updated version
- The old ReplicaSet is scaled down
- The new ReplicaSet is scaled up
- The process happens incrementally

Result:
- Zero (or minimal) downtime
- Continuous availability

### Configuration Parameters

You can control the update behavior using:
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1

maxUnavailable:
- Maximum number of Pods that can be unavailable during the update
- Can be:
  - absolute number (e.g., 1)
  - percentage (e.g., 25%)

maxSurge:
- Maximum number of extra Pods created above the desired count
- Helps speed up rollout

---

## Recreate Strategy

The Recreate strategy deletes all existing Pods before creating new ones.

### How It Works

- All old Pods are terminated
- New Pods are created afterward

Result:
- Causes downtime

When to Use
- Applications that cannot run multiple versions simultaneously
- Legacy systems with strict constraints

## How Rollbacks Work

- Each Deployment update creates a new ReplicaSet
- Old ReplicaSets are retained (by default)
- Rollback scales down the current ReplicaSet
- Scales up the previous one

This makes rollback:
- fast
- reliable
- declarative

---

## Pause and Resume Rollouts

You can pause a rollout to apply multiple changes safely.

Pause
- kubectl rollout pause deployment my-deployment

Resume
- kubectl rollout resume deployment my-deployment

Use case:
- Apply multiple updates before triggering rollout

---

## Troubleshooting Failed Rollouts

Symptoms:
- Pods not becoming Ready
- Rollout stuck
- Deployment not progressing

Steps:
- Check rollout status:
  - kubectl rollout status deployment my-deployment
- Inspect Pods:
  - kubectl get pods
- Check logs:
  - kubectl logs pod-name
- Describe Deployment:
  - kubectl describe deployment my-deployment

Common Issues:
- Image pull errors
- Application crashes
- Failed readiness probes
- Resource limits too low

---

## Key Takeaways

- Deployment manages updates and rollbacks
- RollingUpdate ensures high availability
- Recreate causes downtime
- Each update creates a revision
- Rollbacks are fast and reliable
- kubectl rollout is essential for managing updates

---

## Useful Commands

Check rollout status:
- kubectl rollout status deployment my-deployment

View history:
- kubectl rollout history deployment my-deployment

Rollback:
- kubectl rollout undo deployment my-deployment

Rollback to revision:
- kubectl rollout undo deployment my-deployment --to-revision=2

Pause rollout:
- kubectl rollout pause deployment my-deployment

Resume rollout:
- kubectl rollout resume deployment my-deployment
