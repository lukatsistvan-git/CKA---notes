# Manual Scaling

## Overview

In Kubernetes, scaling means adjusting resources to handle changing workload demands.

There are two main areas:
- Cluster infrastructure scaling
- Workload scaling

---

##  Scaling Cluster Infrastructure

This affects the number or capacity of nodes in the cluster.

### Horizontal Scaling (Adding Nodes)

This means adding more machines (nodes) to the cluster.

#### Manual Scaling

In a kubeadm-based cluster, you can add a node using:
- kubeadm join <control-plane-ip>:6443 --token <token> ...

Result:
- A new node joins the cluster
- More capacity becomes available for scheduling Pods

#### Automatic Scaling

- Cluster Autoscaler automatically adds/removes nodes
- Triggered when Pods cannot be scheduled

### Vertical Scaling (Node Resources)

This means increasing resources of existing nodes:
- CPU
- Memory

#### Manual Approach

- Resize VM or machine
- Restart node if required

Notes:
- Less common in practice
- Horizontal scaling is preferred

---

## Scaling Workloads

This affects the number or size of Pods.

### Horizontal Scaling (Pods)

Increase or decrease the number of identical Pods.

#### Manual Scaling

- kubectl scale deployment myapp --replicas=5

How It Works:
- Deployment updates desired replica count
- ReplicaSet adjusts Pods
- Scheduler places them on nodes

#### Automatic Scaling

- Horizontal Pod Autoscaler (HPA)
- Scales based on CPU/memory usage
- Requires Metrics Server

Example:
- kubectl autoscale deployment myapp --cpu-percent=80 --min=2 --max=5

### Vertical Scaling (Pods)

This means increasing resource limits of existing Pods.

#### Manual Scaling

Edit Deployment:
- kubectl edit deployment myapp

Modify:

resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"

#### Automatic Scaling

- Vertical Pod Autoscaler (VPA)
- Recommends or adjusts resource values
- May restart Pods

---

## Why Scaling Matters

If resources are insufficient:
- Pods remain in Pending state
- Applications become unavailable

Scaling ensures:
- availability
- performance
- efficient resource usage

---

## Key Takeaways

- Scaling happens at two levels:
  - Cluster (nodes)
  - Workloads (Pods)
- Horizontal scaling:
  - add nodes or Pods
- Vertical scaling:
  - increase resources
- Manual scaling:
  - full control
  - requires user intervention
- Automatic scaling:
  - HPA (Pods)
  - Cluster Autoscaler (nodes)
  - VPA (resources)

---

## Useful Commands

Scale Deployment:
- kubectl scale deployment myapp --replicas=5

Edit resources:
- kubectl edit deployment myapp

Check nodes:
- kubectl get nodes

Check pods:
- kubectl get pods
