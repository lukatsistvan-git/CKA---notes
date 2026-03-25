# Control Plane Failure Troubleshooting (kubeadm-focused)

## Overview

The Kubernetes Control Plane manages the entire cluster.

Core components:
* API Server
* Controller Manager
* Scheduler
* etcd (cluster state store)

In **kubeadm-based clusters**, these components run as **static pods**, managed by the **kubelet**.

If the Control Plane fails:
* `kubectl` may stop working
* No scheduling occurs
* Cluster state cannot be modified

---

## Troubleshooting Strategy

### 1. Check if API Server is Reachable

Start with:

kubectl get nodes

* Works → API Server is running
* Fails:
  * connection refused / timeout → API Server down
  * TLS error → certificate issue

Also try:

kubectl get pods -n kube-system

These commands depend on the API Server.  
If they fail → move to **node-level debugging (SSH)**

---

## Step-by-Step Debugging

---

### 2. SSH into Control Plane Node

ssh <control-plane-node>

If HA setup:
* Check **each control plane node individually**

---

## 3. Understand Control Plane Deployment (kubeadm)

Control Plane components run as **static pods**, defined in:
/etc/kubernetes/manifests/

Files:
* kube-apiserver.yaml
* kube-controller-manager.yaml
* kube-scheduler.yaml
* etcd.yaml

The **kubelet automatically manages these pods**

---

## 4. Check kubelet (CRITICAL)

The kubelet is responsible for running static pods.

* sudo systemctl status kubelet

If not running:
* sudo systemctl restart kubelet

Logs:
* sudo journalctl -u kubelet -f

### Common kubelet Issues

* Cannot read manifest files
* Container runtime unavailable
* CNI/network issues
* Certificate problems

---

## 5. Check Control Plane Pods via Container Runtime

Since API Server may be down, use:
* crictl ps

Or (if Docker):
* docker ps

Check logs:
* crictl logs <container-id>

---

## 6. API Server Troubleshooting

Identify API Server container:
* crictl ps | grep kube-apiserver

Logs:
* crictl logs <apiserver-container>

### Common Issues

* Cannot connect to etcd
* Certificate expired or invalid
* Port 6443 already in use
* Misconfiguration

Check port:
* ss -tulnp | grep 6443

---

## 7. Check etcd (CRITICAL DEPENDENCY)

Find etcd container:
* crictl ps | grep etcd

Logs:
* crictl logs <etcd-container>

If external etcd:
* sudo systemctl status etcd

### Health Check

ETCDCTL_API=3 etcdctl endpoint health

### Common etcd Issues

* Disk full
* Data corruption
* Network partition
* Certificate mismatch
* Quorum loss (HA)

If etcd is down → API Server will NOT function

---

## 8. Controller Manager & Scheduler

Check logs:
* crictl logs <controller-manager>
* crictl logs <scheduler>

If failing:
* Pods will not be scheduled
* Controllers (ReplicaSet, Node, etc.) stop working

---

## 9. Check Container Runtime

* sudo systemctl status containerd

Or:

* sudo systemctl status docker

If runtime is down:
* No containers (including Control Plane) will run

---

## 10. Verify Certificates

Certificates are located in:
/etc/kubernetes/pki/

Check expiration:
* kubeadm certs check-expiration

Inspect manually:
* openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

Renew if needed:
* kubeadm certs renew all

---

## 11. Check Network and Ports

Important ports:
* 6443 → API Server
* 2379-2380 → etcd
* 10250 → kubelet

Check:
* ss -tulnp

---

## 12. Static Pod Manifest Issues

Check files:
* ls /etc/kubernetes/manifests/

If missing or corrupted:
* kubelet cannot start Control Plane components

---

## Common Failure Scenarios

### API Server Down

* Cause: etcd unavailable, cert issue, bad config
* Fix:
  * Check logs
  * Verify etcd
  * Fix certificates

---

### etcd Failure

* Cause: disk, corruption, quorum loss
* Fix:
  * Restore from backup
  * Fix infrastructure issue

---

### kubelet Not Running

* Cause: service failure
* Fix:
  * Restart kubelet
  * Check logs

---

### Certificate Expired

* Cause: expired TLS certs
* Fix:
  * kubeadm certs renew

---

### Container Runtime Down

* Cause: containerd/docker stopped
* Fix:
  * Restart runtime

---

## Debugging Workflow Summary

1. kubectl get nodes
2. If fails → SSH to control plane
3. Check kubelet (systemctl + logs)
4. Check containers (crictl ps)
5. API Server logs
6. etcd health
7. Controller/Scheduler logs
8. Container runtime
9. Certificates
10. Network/ports

---

## Best Practices

* Always monitor Control Plane components
* Regularly back up etcd
* Track certificate expiration
* Use HA setup in production
* Centralize logging

---

## Key Takeaways

* API Server is the entry point → check first
* kubelet is critical → manages static pods
* Control Plane runs as static pods in kubeadm
* etcd is the single source of truth
* Most failures are caused by:
  * etcd issues
  * certificate expiration
  * kubelet/runtime failures
