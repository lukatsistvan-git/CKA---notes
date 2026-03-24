# Persistent Volume Claims

## Overview

A **PersistentVolumeClaim (PVC)** is a request for storage made by a user.

Kubernetes will:
- Find a matching **PersistentVolume (PV)** OR
- Dynamically provision one (if a StorageClass is defined)

---

## Example: PersistentVolumeClaim

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
      
---

## How PVC Binding Works

A PVC is bound to a PV when the following conditions are met:
- PV capacity ≥ requested storage
- PV supports requested **accessModes**
- **storageClassName matches** (or both are unset)
- **volumeMode matches** (Filesystem or Block)

---

## Important Behavior

- If a PV is larger than requested → it still binds
- The **entire PV is reserved** for that PVC
- No leftover capacity can be reused by other PVCs

---

## Binding Phases

- **Pending**
  - No matching PV yet

- **Bound**
  - Successfully matched to a PV

- **Lost**
  - Bound PV is no longer available

---

## Volume Modes

Defines how the volume is exposed:
- **Filesystem** *(default)*
- **Block**

Must match between PV and PVC

---

## Access Modes

Defines how the volume can be mounted:

- **ReadWriteOnce (RWO)**
  - Mounted as read-write by a single node

- **ReadOnlyMany (ROX)**
  - Mounted as read-only by multiple nodes

- **ReadWriteMany (RWX)**
  - Mounted as read-write by multiple nodes

- **ReadWriteOncePod (RWOP)** *(Kubernetes 1.22+)*
  - Can be mounted by only **one Pod** in the cluster

> Actual behavior depends on the storage backend.

---

## Reclaim Policy

Defined on the **PersistentVolume**, but affects PVC lifecycle.

- **Retain**
  - Data is preserved
  - Manual cleanup required

- **Delete**
  - PV and underlying storage are deleted automatically

- **Recycle** *(deprecated)*
  - Performs basic cleanup (`rm -rf`)
  - Not recommended

---

## Deleting a PVC

- If a PVC is **in use by a Pod**:
  - It may get stuck in **Terminating**
  - Final deletion happens only after the Pod is removed

---

## Static vs Dynamic Provisioning

### Static

- PV is created manually
- PVC binds to existing PV

---

### Dynamic

- PV is created automatically
- Requires a **StorageClass**

---

## Selectors (Advanced)

PVC can use a selector to match specific PVs:

selector:
  matchLabels:
    type: fast

---

## Common Pitfalls

- StorageClass mismatch
- Access mode mismatch
- VolumeMode mismatch
- Assuming partial PV usage
- PVC stuck in Terminating due to active Pod

---

## Best Practices

- Use **dynamic provisioning**
- Always define **storage requests**
- Avoid manual PV management unless necessary
- Use labels + selectors for fine control

---

## Key Takeaways

- PVC = request for storage
- PV = actual storage resource
- Binding is based on strict matching rules
- One PV → one PVC (no sharing of capacity)
- Dynamic provisioning is preferred

---

## Useful Commands

kubectl get pvc
kubectl describe pvc <name>
kubectl delete pvc <name>
