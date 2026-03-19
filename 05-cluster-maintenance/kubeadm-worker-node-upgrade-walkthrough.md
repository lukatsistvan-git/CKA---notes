# Kubeadm Worker Node Upgrade Walkthrough

## Overview

This guide demonstrates a **step-by-step worker node upgrade** using kubeadm.

Scenario:
- Upgrade from: **v1.31.0**
- Upgrade to: **v1.32.0**
- OS: **Debian/Ubuntu-based system**

---

## Prerequisites

Before starting:
- Control plane is already upgraded
- Node is part of a healthy cluster
- kubectl is configured

---

## Step 0: Drain the Node

Evict workloads safely:
- kubectl drain node01 --ignore-daemonsets --delete-emptydir-data

---

## Step 1: Update Kubernetes Package Repository

Edit the repository file:
- vim /etc/apt/sources.list.d/kubernetes.list

Update to target version:
- deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /

---

## Step 2: Update Package Index

apt update

Fetches latest package metadata from updated repository.

---

## Step 3: Verify Available kubeadm Version

apt-cache madison kubeadm

Example output:
- kubeadm | 1.32.0-1.1 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages

---

## Step 4: Upgrade kubeadm

apt-get install kubeadm=1.32.0-1.1

Notes:
- kubeadm manages cluster upgrade logic
- Must match target Kubernetes version

---

## Step 5: Upgrade the Node Configuration

kubeadm upgrade node

What it does:
- Updates node configuration
- Ensures compatibility with control plane
- Updates local metadata

Important:
- On control plane nodes, use:
  - kubeadm upgrade apply

---

## Step 6: Upgrade kubelet

apt-get install kubelet=1.32.0-1.1

Notes:
- kubelet runs Pods on the node
- Must not be newer than kube-apiserver
- Typically aligned to same version (or x-1)

---

## Step 7: Reload Systemd

systemctl daemon-reload

Required if service definitions changed.

---

## Step 8: Restart kubelet

systemctl restart kubelet

Applies:
- new version
- updated configuration

---

## Step 9: Uncordon the Node

kubectl uncordon node01

Node becomes schedulable again.

---

## Verification

Check node status:
- kubectl get nodes

Check version:
- kubectl get nodes -o wide

Note:
- Displayed version = kubelet version (not kube-apiserver)

---

## Important Notes

- kubeadm does NOT upgrade:
  - kubelet
  - kubectl  
  → must be upgraded manually

- Always:
  - drain before upgrade
  - uncordon after upgrade

- Follow version compatibility rules:
  - kubelet ≤ kube-apiserver
  - recommended: same version

---

## Common Pitfalls

- Forgetting to drain node → workload disruption  
- Not updating apt repo → wrong package version  
- Version mismatch between components  
- Skipping systemctl daemon-reload  
- kubelet not restarted after upgrade  

---

## Summary

- Upgrade is done in **controlled steps**
- kubeadm handles cluster-level logic
- kubelet must be upgraded separately
- drain + uncordon ensures zero downtime (with replicas)

---

## Reference

Official documentation:
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
