# Worker Node Failure Troubleshooting

## Overview

Worker nodes are responsible for running application workloads (Pods).

Key components on a worker node:
* kubelet (node agent)
* kube-proxy (networking)
* Container runtime (containerd / Docker)
* OS resources (CPU, memory, disk, network)

If a worker node fails:
* Pods may stop running or become unreachable
* Node may become `NotReady` or `Unknown`
* Workloads may be rescheduled to other nodes

---

## Troubleshooting Strategy

### 1. Check Node Status

Start with:
* kubectl get nodes

### Common STATUS values

* `Ready` → Node is healthy
* `NotReady` → Node has issues
* `Unknown` → Control Plane cannot reach node

If a node is not `Ready` → investigate further

---

## Step-by-Step Debugging

---

### 2. Describe the Node

* kubectl describe node <node-name>

### What to Check

* **Conditions**
* **Events**
* **Taints**
* **Resource pressure warnings**

---

### Node Conditions Explained

* **Ready**
  * `True` → healthy
  * `False` → node not functioning

* **MemoryPressure**
  * `True` → low memory, risk of OOM kills

* **DiskPressure**
  * `True` → low disk space

* **PIDPressure**
  * `True` → too many processes

* **NetworkUnavailable**
  * `True` → networking/CNI issue

* **OutOfDisk** (legacy)
  * `True` → disk completely full

Note:
* `False` is healthy for all except `Ready`

---

## 3. Check Pods on the Node

* kubectl get pods -o wide

Filter by node:

* kubectl get pods --field-selector spec.nodeName=<node-name>

Look for:
* Pods stuck in `Terminating`
* Pods not rescheduled
* Frequent restarts

---

## 4. SSH into the Worker Node

* ssh <worker-node>

---

## 5. Check kubelet (CRITICAL)

kubelet is responsible for:
* Communicating with API Server
* Managing Pods on the node

Check status:

* sudo systemctl status kubelet

Logs:
* sudo journalctl -u kubelet

### Common kubelet Issues

* Cannot reach API Server
* Certificate problems
* CNI/network errors
* cgroup or runtime issues
* Repeated restarts

---

## 6. Check Container Runtime

Check runtime status:
* sudo systemctl status containerd

Or:

* sudo systemctl status docker

If runtime is down:
* Pods will not run

---

## 7. Check Node Resources (OS Level)

### CPU & Memory

* top

Look for:
* High CPU usage
* Memory exhaustion

---

### Disk Usage

* df -h

Look for:
* Full root (`/`) or `/var`
* No available space for containers

---

## 8. Check Networking

### Symptoms

* Node shows `NotReady`
* Pods cannot communicate
* `NetworkUnavailable = True`

### Actions

* Verify CNI plugin is running
* Check kube-proxy:

* sudo systemctl status kube-proxy

* Test connectivity:
  * ping other nodes
  * curl cluster services

---

## 9. Check kubelet Certificates

Certificates are located in:
/var/lib/kubelet/pki/

Check certificate:
* openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout

### What to Verify

* **Not After** → expiration date
* **Subject** → matches node name
* **Issuer** → correct CA

### Notes

* kubelet usually auto-renews certificates
* Renewal fails if:
  * API Server is unreachable
  * bootstrap token expired

---

## 10. Check Node Events

* kubectl get events --sort-by=.metadata.creationTimestamp

Look for:
* Node disconnects
* Resource pressure warnings
* Kubelet errors

---

## Common Failure Scenarios

### Node NotReady

* Cause:
  * kubelet stopped
  * network issue
* Fix:
  * restart kubelet
  * verify connectivity

---

### MemoryPressure / DiskPressure

* Cause:
  * resource exhaustion
* Fix:
  * free resources
  * adjust limits

---

### kubelet Cannot Reach API Server

* Cause:
  * network issue
  * certificate problem
* Fix:
  * verify network
  * check cert validity

---

### Container Runtime Down

* Cause:
  * containerd/docker stopped
* Fix:
  * restart runtime

---

### Networking (CNI) Failure

* Cause:
  * CNI plugin crash/misconfig
* Fix:
  * check CNI pods/logs
  * verify network setup

---

## Debugging Workflow Summary

1. kubectl get nodes
2. kubectl describe node
3. Check pods on node
4. SSH into node
5. Check kubelet (status + logs)
6. Check container runtime
7. Check CPU/memory/disk (top, df -h)
8. Verify networking (CNI, kube-proxy)
9. Check certificates
10. Review events

---

## Best Practices

* Monitor node health continuously
* Set proper resource requests/limits
* Use node auto-repair in production
* Keep kubelet and runtime stable
* Ensure sufficient disk and memory capacity

---

## Key Takeaways

* kubelet is the most critical component on a worker node
* Most node issues are caused by:
  * resource exhaustion
  * network problems
  * kubelet failures
* Node conditions provide quick health insight
* OS-level checks are essential for root cause analysis
