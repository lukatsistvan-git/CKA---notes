# Cluster Upgrade Process

## Overview

Upgrading a Kubernetes cluster requires:
- strict **version compatibility awareness**
- a **step-by-step upgrade strategy**
- minimizing downtime and risk

---

## Version Compatibility Rules

### 1. kube-apiserver (Central Component)

The **kube-apiserver** is the reference point.

All other components must be compatible with it.

---

### 2. Controller Manager & Scheduler

Constraints:
- Cannot be newer than kube-apiserver

Supported versions:
- Same version (recommended)
- One version lower

Example:
- kube-apiserver v1.28  
  → controller-manager v1.28 or v1.27

---

### 3. kubelet & kube-proxy

Constraints:
- Cannot be newer than kube-apiserver

More flexible compatibility:
- Up to **2 versions older (x-2)**

Example:
- kube-apiserver v1.28  
  → kubelet v1.26 is still supported

---

### 4. kubectl (CLI)

kubectl is the most flexible:

Supported:
- x-1, x, or x+1

Example:
- kube-apiserver v1.28  
  → kubectl v1.27, v1.28, or v1.29

---

### 5. etcd & CoreDNS

- Have their own versioning
- Must match Kubernetes requirements

Important:
- Each Kubernetes version requires **specific etcd and CoreDNS versions**
- Always check official compatibility matrix

---

## Upgrade Strategy (Step-by-Step)

### Never Skip Minor Versions

Kubernetes only supports **incremental upgrades (x → x+1)**.

Example:
v1.10 → v1.11 → v1.12 → v1.13

Why:
- API deprecations
- behavioral changes
- internal system updates

Tools like:
- kubeadm  
only support stepwise upgrades

---

## Upgrade Order

Always upgrade in this order:

1. **Control Plane (Master)**
2. **Worker Nodes**

---

## Control Plane Upgrade Behavior

During control plane upgrade:

### Not Available:
- Creating new Pods / Deployments
- Scheduling (scheduler down)
- Controllers (no self-healing, autoscaling)
- kubectl operations (if API server unavailable)

### Still Working:
- Existing Pods continue running on worker nodes
- kubelet manages containers locally
- Services (ClusterIP, NodePort) continue functioning
- Application traffic continues if replicas exist

---

## Worker Node Upgrade Strategies

### 1. All-at-Once (Parallel Upgrade)

Process:
- Upgrade all nodes simultaneously

Disadvantages:
- High risk of downtime
- Pods may be lost
- Hard to troubleshoot failures

Use only if:
- non-production (dev / staging)
- stateless workloads with high redundancy

---

### 2. Rolling Upgrade (Recommended)

Process (per node):

1. Drain node:

kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

2. Upgrade node:
- OS
- kubelet
- kube-proxy

3. Uncordon node:
- kubectl uncordon <node>

Repeat for each node.

---

### Advantages

- No downtime (with proper replicas)
- Controlled, predictable process
- Easy troubleshooting
- Minimal risk

---

### 3. Blue/Green (Replace Node Strategy)

Process:
1. Create new nodes with updated version
2. Drain old node:
- kubectl drain <old-node> --ignore-daemonsets --delete-emptydir-data
3. Pods are rescheduled to new nodes
4. Validate:
- testing
- monitoring
5. Remove old node from cluster

---

### Advantages

- Clean environment
- Safer for major infrastructure changes
- Works well with automation (e.g. cloud environments)

---

## Important Notes

- Always upgrade **control plane first**
- Ensure **backups (etcd)** before upgrade
- Validate workloads after each step
- Maintain **high availability (replicas)**
- Avoid skipping versions

---

## Troubleshooting

Upgrade failed?

Check:
- Node versions:
  - kubectl get nodes -o wide

Check component status:
- kubectl get componentstatuses

Check logs:
- kubelet
- control plane components

Check events:
- kubectl get events

---

## Key Takeaways

- kube-apiserver defines compatibility
- Never skip minor versions
- Upgrade order:
  - control plane → worker nodes
- Rolling upgrade is safest
- Blue/Green is modern and scalable
- Parallel upgrade is risky

---

## Useful Commands

Check cluster version  
- kubectl version

Check nodes  
- kubectl get nodes

Drain node  
- kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

Uncordon node  
- kubectl uncordon <node>

Upgrade (kubeadm example)  
- kubeadm upgrade plan  
- kubeadm upgrade apply <version>
