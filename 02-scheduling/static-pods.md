# Static Pods

## Overview

A **Static Pod** is a Pod that is managed directly by the **kubelet**, not by the Kubernetes control plane.
- No Deployment
- No ReplicaSet
- No Scheduler involvement

---

## How It Works

The kubelet:
- watches a specific directory on the Node
- reads Pod definition files (`.yaml`, `.json`)
- automatically creates and manages Pods

Default location:
/etc/kubernetes/manifests/

The directory is configured via kubelet:
--pod-manifest-path=/path/to/manifests

Or in kubelet config file:
/var/lib/kubelet/config.yaml

Behavior
- If a file is added → Pod is created
- If a file is modified → Pod is updated
- If a file is removed → Pod is deleted

---

## Mirror Pods

Although Static Pods are not created via the API server:
- kubelet creates a mirror Pod
- visible via kubectl

Important:
- Mirror Pods are read-only representations.

---

## Use Cases

Static Pods are commonly used for critical system components:
- kube-apiserver
- kube-controller-manager
- etcd

---

## Important Notes

- Cannot be managed with:
  - Deployment
  - ReplicaSet
- Not suitable for scaling
- Used for node-level control

Deleting Static Pods
- This will NOT work:
  - kubectl delete pod static-nginx
- Correct way:
  - rm /etc/kubernetes/manifests/static-nginx.yaml

Naming
Mirror Pod name format:
- <pod-name>-<node-name>
Example:
- static-nginx-node1

---

## Troubleshooting

Pod not starting?

Check:
- journalctl -u kubelet

Check manifest directory
- ls /etc/kubernetes/manifests/

Check running Pods
- kubectl get pods -o wide

Identify Static Pods
- kubectl get pods -o yaml | grep -i static

---

## Key Takeaways

- Static Pods are managed directly by the kubelet
- Defined via local files on the Node
- Do NOT require:
  - scheduler
  - control plane
- Visible as mirror Pods
- Commonly used for:
  - control plane components
