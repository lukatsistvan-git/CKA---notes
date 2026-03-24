# Persistent Volumes

## Overview

The Kubernetes storage model is designed to **decouple storage from the pod lifecycle**.

There are three main components:
- **Volume** → pod-level storage
- **PersistentVolume (PV)** → cluster-level storage resource
- **PersistentVolumeClaim (PVC)** → user request for storage

---

## Container Storage Interface (CSI)

### Concept

The **Container Storage Interface (CSI)** is a standard API that allows Kubernetes to work with different storage systems in a unified way.

### Purpose

- Decouple Kubernetes from storage providers
- Enable vendor-neutral storage integration
- Provide a plugin-based architecture

---

## How it Works

- CSI is a **gRPC-based API**
- Storage providers implement **CSI drivers**
- Kubernetes communicates with storage through these drivers

---

## Supported Operations

- Create / Delete volumes
- Attach / Detach to nodes
- Mount / Unmount to pods
- Snapshot / Restore (if supported)

---

## Important Note

- Legacy in-tree drivers (e.g., awsElasticBlockStore, gcePersistentDisk) are **deprecated**
- Modern approach: use **CSI drivers**

---

## Volumes

### Concept

A **Volume** is a storage abstraction that:
- Is attached to a Pod
- Can be shared between containers in the same Pod
- Survives container restarts
- Is typically deleted when the Pod is deleted

---

## Volume Types

### 1. Ephemeral Volumes

Tied to the Pod lifecycle.

#### Examples

- **emptyDir**
  - Created empty when the Pod starts

- **configMap**
  - Configuration exposed as files

- **secret**
  - Sensitive data exposed as files

- **downwardAPI**
  - Pod metadata exposed as files

---

### 2. Persistent Volumes (via PVC)

Persist beyond Pod restarts.

#### Examples

- **hostPath** *(dev / single node only)*
- **nfs**
- **CSI-based volumes** *(recommended)*

---

## PersistentVolume (PV)

### Concept

A **PersistentVolume (PV)** is a cluster-level resource that:
- Represents physical or network storage
- Is created by an admin or dynamically
- Exists independently of Pods

---

## PV Lifecycle

- **Available**
- **Bound**
- **Released**
- **Failed**

---

## Key PV Fields

- **capacity**
- **accessModes**
- **persistentVolumeReclaimPolicy**
- **storageClassName**
- **volumeMode**

---

## Access Modes

- **ReadWriteOnce (RWO)**
  - Mounted as read-write by a single node

- **ReadOnlyMany (ROX)**
  - Mounted as read-only by multiple nodes

- **ReadWriteMany (RWX)**
  - Mounted as read-write by multiple nodes

Availability depends on the underlying storage backend.

---

## Reclaim Policy

Defines what happens after a PVC is deleted:

- **Retain**
  - Data is preserved (manual cleanup required)

- **Delete**
  - Storage is automatically deleted

- **Recycle** *(deprecated)*

---

## Volume Binding

### Static Provisioning

- Admin manually creates PVs
- PVC binds to an existing PV

---

### Dynamic Provisioning

- PV is automatically created when a PVC is requested
- Requires a **StorageClass**

---

## Example: PersistentVolume

apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  nfs:
    path: /exports/data
    server: 192.168.1.100

---

## Best Practices

- Use **CSI drivers**
- Avoid `hostPath` in production
- Use **StorageClass + dynamic provisioning**
- Always define a reclaim policy

---

## Common Pitfalls

- Misunderstanding access modes
- Using hostPath in production
- Missing StorageClass for dynamic provisioning
- PV/PVC mismatch

---

## Key Takeaways

- PV = storage resource
- PVC = storage request
- CSI = modern storage integration
- Dynamic provisioning is recommended

---

## Useful Commands

- kubectl get pv
- kubectl get pvc
- kubectl describe pv <name>
- kubectl describe pvc <name>
