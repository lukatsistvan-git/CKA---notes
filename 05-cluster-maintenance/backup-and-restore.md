# Backup and Restore

## Overview

Backing up a Kubernetes cluster is critical for:
- disaster recovery
- data protection
- cluster migration

There are two main approaches:

1. **Resource-based backup (YAML)**
2. **etcd database snapshot (full backup)**

---

## 1. Resource Backup (YAML)

### Concept

Export Kubernetes resources as YAML manifests using kubectl.

Example:

kubectl get all --all-namespaces -o yaml > all-resources.yaml

---

### Limitations

This does NOT include everything:
- Secrets (not always safely reusable)
- ConfigMaps (must be explicitly included)
- PersistentVolumes (data is external)
- Custom Resources (CRDs)
- Cluster state metadata

Better approach:
- kubectl get configmaps --all-namespaces -o yaml  
- kubectl get secrets --all-namespaces -o yaml  
- kubectl get pvc --all-namespaces -o yaml  

---

### Best Practice

Use **GitOps**:

- Store manifests in Git
- Version control changes
- Recreate cluster declaratively

---

## 2. etcd Backup (Full Cluster Backup)

### Concept

etcd stores the **entire cluster state**:
- Pods
- Deployments
- Secrets
- ConfigMaps
- RBAC
- All API objects

This is the **most complete backup method**.

---

## Important: ETCDCTL_API

Before using etcdctl:

- export ETCDCTL_API=3
or
- ETCDCTL_API=3 etcdctl snapshot save ...

Why:
- Kubernetes uses **etcd v3 API**
- Required for:
  - snapshot save
  - snapshot restore
  - snapshot status

Without this:
- commands may fail or behave incorrectly

---

## Taking an etcd Snapshot

### Basic Command

etcdctl snapshot save snapshot.db

---

### Check Snapshot Status

etcdctl snapshot status snapshot.db

---

### Production (TLS-secured etcd)

etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key

---

## Restore Process (kubeadm-based cluster)

### Step 1: Stop API Server

In kubeadm clusters, the API server runs as a **static Pod**.

mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

---

### Step 2: Restore etcd Snapshot

etcdctl snapshot restore snapshot.db \
  --data-dir=/var/lib/etcd-from-backup

---

### Step 3: Update etcd Configuration

Edit:

/etc/kubernetes/manifests/etcd.yaml

Update:

- --data-dir=/var/lib/etcd-from-backup

---

### Step 4: Restart etcd

Handled automatically by kubelet (static Pod).

---

### Step 5: Restore API Server

mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

---

### Step 6: Verify Cluster

- kubectl get nodes  
- kubectl get pods -A  

---

## When to Use Which?

### YAML Backup

- Dev / staging
- GitOps workflows
- Partial restore

---

### etcd Snapshot

- Production clusters
- Full disaster recovery
- Critical systems

---

## Important Notes

- etcd backup = **only full backup**
- Always backup before upgrades
- Store backups off-cluster
- Regularly test restore

---

## Common Pitfalls

- Missing ETCDCTL_API=3  
- Forgetting TLS flags  
- Assuming YAML backup is complete  
- Wrong data-dir on restore  
- Not updating etcd manifest  

---

## Key Takeaways

- YAML = partial backup
- etcd = full backup
- Use both in production
- ETCDCTL_API=3 is required

---

## Useful Commands

export ETCDCTL_API=3  

kubectl get all --all-namespaces -o yaml  

etcdctl snapshot save snapshot.db  

etcdctl snapshot status snapshot.db  

etcdctl snapshot restore snapshot.db  
