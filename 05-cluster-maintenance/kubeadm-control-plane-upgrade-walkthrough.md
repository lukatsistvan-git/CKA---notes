# Kubeadm Control Plane Upgrade Walkthrough

## Overview

This guide demonstrates a **step-by-step control plane upgrade** using kubeadm.

Scenario:
- Upgrade from: **v1.31.0**
- Upgrade to: **v1.32.0**
- OS: **Debian/Ubuntu-based system**

---

## Prerequisites

Before starting:
- Backup of etcd is strongly recommended
- Cluster is healthy
- kubectl is configured
- Worker nodes are ready to handle workloads

---

## Step 0: Drain the Control Plane Node

- kubectl drain controlplane --ignore-daemonsets --delete-emptydir-data

Notes:
- Ensures workloads are moved to worker nodes
- DaemonSets are ignored

---

## Step 1: Update Kubernetes Package Repository

Edit the repository file:
- vim /etc/apt/sources.list.d/kubernetes.list

Example (before):
- deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /

Update to target version:
- deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /

---

## Step 2: Update Package Index

apt update

Fetch latest package metadata.

---

## Step 3: Verify Available kubeadm Version

apt-cache madison kubeadm

Example:
- kubeadm | 1.32.0-1.1 | https://apt.kubernetes.io ...

---

## Step 4: Upgrade kubeadm

apt-get install kubeadm=1.32.0-1.1

Notes:
- Upgrades only the kubeadm binary
- Required before running upgrade commands

---

## Step 5: Plan the Upgrade

kubeadm upgrade plan v1.32.0

What it does:
- Checks cluster state
- Verifies upgrade compatibility
- Shows required component upgrades
- Highlights potential issues

---

## Step 6: Apply the Upgrade

kubeadm upgrade apply v1.32.0

This step upgrades:
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- etcd

Also updates:
- static Pod manifests (/etc/kubernetes/manifests)

Important:
- This step runs ONLY on control plane nodes

---

## Step 7: Upgrade kubelet

apt-get install kubelet=1.32.0-1.1

Notes:
- kubeadm does NOT upgrade kubelet automatically
- kubelet must match control plane version (or x-1)

---

## Step 8: Reload Systemd

systemctl daemon-reload

Reloads updated service configurations.

---

## Step 9: Restart kubelet

systemctl restart kubelet

Applies new version and configuration.

---

## Step 10: Uncordon the Node

kubectl uncordon controlplane

Node becomes schedulable again.

---

## Verification

Check cluster:
- kubectl get nodes

Check versions:
- kubectl get nodes -o wide

Check control plane Pods:
- kubectl get pods -n kube-system

---

## Important Notes

- Always upgrade **control plane first**
- kubeadm handles:
  - control plane components
- kubelet must be upgraded manually
- kubectl should also be updated separately

During upgrade:
- Control plane may be temporarily unavailable
- Existing workloads continue running on worker nodes

---

## Common Pitfalls

- Skipping kubeadm upgrade plan  
- Version mismatch between components  
- Forgetting kubelet upgrade  
- Not restarting kubelet  
- No etcd backup before upgrade  

---

## Summary

- Control plane upgrade is the **first step**
- kubeadm manages core component upgrades
- kubelet upgrade is manual
- Proper sequencing ensures cluster stability

---

## Reference

Official documentation:

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
