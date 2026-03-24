# Storage Classes

## Overview

A **StorageClass** defines how a **PersistentVolume (PV)** should be dynamically provisioned.

It is used when a **PersistentVolumeClaim (PVC)** requests storage and no suitable PV already exists.

---

## Purpose

- Enable **dynamic provisioning** of volumes
- Define storage characteristics (performance, type, replication)
- Support multiple storage backends in a single cluster

Examples:
- SSD vs HDD
- Local vs cloud storage
- Different CSI drivers

---

## Example: StorageClass

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true

---

## Key Fields

### provisioner

- Defines which volume plugin is used
- Usually a **CSI driver**

Examples:
- `ebs.csi.aws.com`
- `pd.csi.storage.gke.io`
- `disk.csi.azure.com`

Legacy provisioners like `kubernetes.io/aws-ebs` are deprecated.

---

### parameters

- Backend-specific configuration
- Passed to the storage provider

Examples:
- disk type (ssd, hdd)
- filesystem type
- replication settings

---

### reclaimPolicy

Same as PV reclaim policy:
- **Delete** (default for dynamic provisioning)
- **Retain**

---

### volumeBindingMode

Controls **when the PV is provisioned and bound**.

- **Immediate**
  - PV is created as soon as PVC is created

- **WaitForFirstConsumer**
  - PV is created only when a Pod uses the PVC
  - Ensures correct **node / zone placement**

Strongly recommended for multi-zone clusters.

---

### allowVolumeExpansion

- Allows resizing of PVCs
- Must also be supported by the storage backend

---

## Default StorageClass

A cluster can have a **default StorageClass**.

- Used when PVC does not specify `storageClassName`
- Defined via annotation:

metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"

---

## How It Works

1. User creates a PVC
2. PVC references a StorageClass
3. Kubernetes calls the **provisioner**
4. A new PV is created automatically
5. PVC is bound to the new PV

---

## StorageClass and PVC

PVC example referencing a StorageClass:

spec:
  storageClassName: fast-ssd

---

## Common Pitfalls

- Using deprecated in-tree provisioners
- Missing default StorageClass
- Forgetting `WaitForFirstConsumer` in multi-zone clusters
- Backend does not support volume expansion
- Mismatch between StorageClass and PVC

---

## Best Practices

- Use **CSI-based provisioners**
- Always define a **default StorageClass**
- Use `WaitForFirstConsumer` for production workloads
- Enable `allowVolumeExpansion` when possible
- Use meaningful naming (e.g., fast-ssd, slow-hdd)

---

## Key Takeaways

- StorageClass = template for dynamic PV provisioning
- Works together with PVC
- Enables automation and flexibility
- CSI is the modern standard

---

## Useful Commands

- kubectl get storageclass
- kubectl get sc
- kubectl describe storageclass <name>
