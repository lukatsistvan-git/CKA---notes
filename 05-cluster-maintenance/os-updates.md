# OS Updates

## Overview

Operating system (OS) upgrades on Kubernetes nodes require **careful handling** to avoid application downtime.

Key concern:
- What happens to Pods when a Node goes down?

---

## What Happens When a Node Goes Down?

When a Node becomes unavailable:
- All Pods running on that Node become **inaccessible**

If the application is replicated:
- Other Pod instances (on different Nodes) continue serving traffic

If NOT replicated:
- The application becomes unavailable until:
  - the Node returns, or
  - the Pod is recreated on another Node (if it is possible)

---

## Pod Eviction Timeout

This behavior is controlled by the **kube-controller-manager**.

Default:
- **5 minutes (300 seconds)**

This is often referred to as:
- `pod-eviction-timeout`

---

## Behavior Within Eviction Timeout

If the Node comes back **before the timeout expires**:
- kubelet reconnects to the API Server
- Node status becomes **Ready**
- Existing Pods are **NOT deleted**
- Pods can return to **Running state automatically** (if dependencies recover)

Important:
- No rescheduling happens
- Controllers (ReplicaSet, Deployment) do NOT create new Pods

---

## Behavior After Eviction Timeout

If the Node is down **longer than 5 minutes**:
- Node is marked as **NotReady / dead**
- Kubernetes evicts:
  - all non-static Pods
  - all non-DaemonSet Pods

Then:
- Controllers (e.g. ReplicaSet, Deployment) recreate Pods on other Nodes
- Original Pods are considered **lost permanently**

---

## Node Returns After Timeout

If the Node comes back **after eviction**:
- Previously running Pods do NOT come back
- The Node is effectively **empty (blank)**

New Pods will only be scheduled if:
- the Node is marked schedulable again
- the scheduler decides to place Pods there

---

## Safe Node Maintenance (OS Upgrade Workflow)

To safely upgrade a Node:

### 1. Cordon the Node

Prevents new Pods from being scheduled:
- kubectl cordon <node-name>

---

### 2. Drain the Node

Safely evicts Pods and reschedules them:
- kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

Notes:
- ignores DaemonSet Pods (they stay)
- deletes Pods using emptyDir volumes

---

### 3. Perform OS Upgrade

- apply updates
- reboot if necessary

---

### 4. Uncordon the Node

Make Node schedulable again:
- kubectl uncordon <node-name>

---

## Important Notes

- Always **drain before maintenance**
- DaemonSet Pods are NOT evicted automatically
- Static Pods are managed by kubelet, not controllers
- Applications should use **replicas** to avoid downtime
- Draining ensures controlled rescheduling instead of waiting for eviction timeout

---

## Troubleshooting

Node stuck in NotReady?

Check:
- kubelet status
- node conditions:
  - kubectl describe node <node-name>

Check Pods:
- kubectl get pods -o wide

Check events:
- kubectl get events

---

## Key Takeaways

- Node failure makes Pods unavailable
- Default eviction timeout: **5 minutes**
- Before timeout:
  - Pods stay, no rescheduling
- After timeout:
  - Pods are evicted and recreated elsewhere
- Returning Node after timeout is **empty**
- Use:
  - cordon
  - drain
  - uncordon  
  for safe OS upgrades

---

## Useful Commands

Cordon Node  
- kubectl cordon <node-name>

Drain Node  
- kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

Uncordon Node  
- kubectl uncordon <node-name>

Check Nodes  
- kubectl get nodes

Describe Node  
- kubectl describe node <node-name>
