# Kubernetes Architecture

## Overview

A Kubernetes cluster consists of two main parts:

- **Control Plane** (manages the cluster)
- **Worker Nodes** (run applications)

High-level architecture:
Control Plane
- API Server
- etcd
- Controller Manager
- Scheduler

Worker Node
- Kubelet
- Kube-proxy
- Container Runtime

---

## Control Plane Components

The **Control Plane** is responsible for managing the cluster and maintaining the desired state.

---

### etcd

**etcd** is a distributed key-value store used as the **database of Kubernetes**.

It stores:
- cluster state
- configuration data
- all Kubernetes objects (Pods, Nodes, Secrets, etc.)

Key characteristics:
- uses the **Raft consensus algorithm**
- ensures **data consistency**
- listens on port **2379 (TCP)**

etcdctl is a command-line tool to interact with etcd:

---

### kube-apiserver

The API Server is the entry point to the Kubernetes cluster.

Responsibilities:
- handles all REST requests
- authentication and authorization
- request validation
- communication with etcd

Important:
- It is the only component that communicates directly with etcd.

All other components interact through the API server.

---

### kube-controller-manager

Runs multiple controllers that continuously monitor the cluster state.

Examples:
- Node Controller
- ReplicaSet Controller
- Deployment Controller
- Namespace Controller
- Job Controller

Controllers work on the principle:
- Desired state ≠ Current state → Take action

Example:

Desired Pods: 3
Running Pods: 2
→ Controller creates 1 new Pod

---

### kube-scheduler

The Scheduler decides which node a Pod should run on.

It evaluates:
- resource requirements (CPU, memory)
- node capacity
- labels and selectors
- taints and tolerations
- affinity / anti-affinity rules
- priorities

Important:
- The Scheduler only assigns a node to a Pod. It does NOT start the Pod.

---

## Worker Node Components

Worker nodes run the actual application workloads.

---

### kubelet

The kubelet is the primary agent on each worker node.

Responsibilities:
- registers the node with the cluster
- communicates with the API server
- ensures containers are running as expected
- monitors Pod status

Important:
- kubelet does NOT create Pods by itself
- it follows instructions from the API server

Note:
- kubeadm does NOT install kubelet automatically — it must be installed separately.

---

### kube-proxy

The kube-proxy handles networking for Services.

Responsibilities:
- watches Services and Endpoints via API server
- creates networking rules (iptables / IPVS)
- enables communication between Services and Pods

Function:
- Service IP → kube-proxy → Pod IP

Provides:
- internal load balancing
- Service routing

---

### Container Runtime

The container runtime is responsible for running containers.

Examples:
- containerd
- CRI-O

Responsibilities:
- pull images
- start/stop containers
- manage container lifecycle

---

## How Components Work Together

Example for creating a Pod:

User → API Server → etcd (store desired state)
            ↓
     Scheduler assigns Node
            ↓
     kubelet creates Pod
            ↓
     kube-proxy enables networking

---

## Key Takeaways
- The Control Plane manages the cluster
- The API Server is the central communication hub
- etcd stores all cluster data
- The Scheduler assigns Pods to nodes
- The Controller Manager maintains desired state
- The kubelet runs Pods on nodes
- The kube-proxy handles networking
