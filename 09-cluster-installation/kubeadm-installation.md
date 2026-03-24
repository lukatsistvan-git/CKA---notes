# kubeadm Installation

## Overview

This section covers how to **design and bootstrap a Kubernetes cluster using kubeadm**, based on Kubernetes best practices.

`kubeadm` is a tool that helps you:
* Initialize a Kubernetes control plane
* Join worker nodes to the cluster
* Follow production-grade setup patterns

> kubeadm does **not provision infrastructure** — it configures Kubernetes on existing machines.

---

## Cluster Design Considerations

Before installing a cluster, you must define key requirements.

---

### Purpose of the Cluster

* **Learning / Development**
  
  * Single node or minimal setup
  * Simpler networking and security

* **Production**

  * High availability (HA)
  * Strong security (RBAC, NetworkPolicy)
  * Monitoring and backups

---

### Hosting Platform

* **Cloud (Managed Kubernetes)**

  * Examples: EKS, GKE, AKS
  * Easier scaling and HA
  * Less operational overhead

* **On-Premise (kubeadm)**

  * Full control
  * More operational responsibility
  * Requires manual networking, storage, HA setup

---

### Workloads

Consider:
* Stateless vs stateful applications
* CPU / memory intensive workloads
* GPU requirements
* Batch jobs vs real-time services

This affects:
* Node sizing
* Storage configuration
* Scheduling policies

---

### Scale and Resource Planning

* Number of applications
* Expected traffic
* Growth over time

Impacts:
* Node count
* Autoscaling strategy
* Network design

---

### Networking Considerations

* Pod CIDR range (e.g., `10.244.0.0/16`)
* Service CIDR range (e.g., `10.96.0.0/12`)
* CNI plugin choice
* Ingress / Gateway setup

---

## High Availability (HA) Design

### Control Plane Redundancy

* Single control plane = **Single Point of Failure (SPOF)**
* Recommended:
  * **At least 3 control plane nodes**

---

### Load Balancer

* Required for HA clusters
* Distributes traffic to API servers

Example:
Client → Load Balancer → API Servers


---

### etcd Cluster

* Stores all cluster state
* Distributed key-value store

#### Requirements

* Minimum **3 nodes**
* Uses **quorum-based consensus**

---

### Quorum Concept

* Majority must be available

Examples:
* 3 nodes → quorum = 2
* 5 nodes → quorum = 3

If quorum is lost → cluster becomes read-only/unavailable

---

### Leader Election (RAFT)

* etcd uses **RAFT consensus algorithm**
* One node is **leader**
* Others are **followers**

Behavior:
* Leader handles writes
* Followers replicate data
* New leader elected automatically on failure

---

### etcd Topology

Two options:

#### Stacked etcd (default in kubeadm)

* etcd runs on control plane nodes

Pros:
* Simpler setup

Cons:
* Shared failure domain

---

#### External etcd

* Runs on separate nodes

Pros:
* Better isolation
* Improved reliability

Cons:
* More complex setup

---

## kubeadm Architecture

kubeadm sets up:

### Control Plane Components

* kube-apiserver
* kube-controller-manager
* kube-scheduler
* etcd
* CoreDNS

---

### Node Components

* kubelet
* kube-proxy
* Container runtime (containerd, CRI-O)

---

## Installation Prerequisites

All nodes must have:
* Linux OS
* Container runtime installed
* Swap **disabled**
* Unique hostname and IP
* Network connectivity between nodes

---

### Required Tools

Install on all nodes:
* kubeadm
* kubelet
* kubectl

---

## Initialize Control Plane

Run on the first control plane node:
* kubeadm init

Optional (custom CIDR):
* kubeadm init --pod-network-cidr=10.244.0.0/16

---

### Output

* kubeconfig file instructions
* `kubeadm join` command for worker nodes

---

## Configure kubectl

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config


---

## Install CNI Plugin

Cluster networking is not functional until a CNI plugin is installed.

Example (Flannel):

kubectl apply -f <flannel-yaml>

---

## Join Worker Nodes

Run on worker nodes:

kubeadm join <control-plane-ip>:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>

---

## Add Additional Control Plane Nodes (HA)

kubeadm join ... --control-plane

Requires:
* Shared certificates
* Load balancer endpoint

---

## Verification

Check nodes:
* kubectl get nodes

Check system pods:
* kubectl get pods -n kube-system

---

## Common Pitfalls

* Forgetting to disable swap
* Missing CNI plugin → Pods stuck in `Pending`
* Incorrect Pod CIDR
* Firewall blocking required ports
* Not using a load balancer in HA setups

---

## Best Practices

* Use **containerd** as runtime
* Always install a CNI plugin immediately
* Use odd number of etcd nodes (3, 5)
* Separate environments (dev / staging / prod)
* Backup etcd regularly

---

## Key Takeaways

* kubeadm is the standard tool for manual cluster setup
* Proper design is critical before installation
* HA requires multiple control plane nodes and etcd quorum
* Networking must be configured via CNI
* kubeadm simplifies but does not automate infrastructure
