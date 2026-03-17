# DaemonSets

## Overview

A **DaemonSet** ensures that a specific Pod runs on:
- every Node  
or  
- a subset of Nodes (based on rules)
Each Node gets **exactly one Pod instance**.

---

## Use Cases

DaemonSets are typically used for **node-level infrastructure tasks**:
- logging agents (e.g. Fluentd, Logstash)
- monitoring agents (e.g. Prometheus Node Exporter)
- networking components (CNI plugins)
- storage daemons

---

## How It Works

The DaemonSet controller:
- creates one Pod per eligible Node
- automatically adds Pods when new Nodes join
- removes Pods when Nodes are removed

---

## Example

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemon
spec:
  selector:
    matchLabels:
      app: my-daemon
  template:
    metadata:
      labels:
        app: my-daemon
    spec:
      containers:
      - name: my-daemon-container
        image: my-image

## Scheduling Behavior

Important:
- DaemonSets do NOT behave like Deployments.
- Scheduler is still involved
- BUT Pod placement is predefined per Node

Effectively:
- Kubernetes ensures 1 Pod per Node
- Not based on replica count

## Controlling Where Pods Run

### Node Selector

nodeSelector:
  disktype: ssd

### Node Affinity

affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd

### Tolerations

Useful for scheduling on tainted Nodes (e.g. control-plane):

tolerations:
- key: "node-role.kubernetes.io/control-plane"
  operator: "Exists"
  effect: "NoSchedule"

## Important Notes

- Not designed for scaling workloads
- Automatically adapts to cluster size
- Typically used for system-level services
- Runs in background on every Node

## Troubleshooting

Missing Pod on a Node?

Check:
- kubectl get daemonsets

Check DaemonSet status
- kubectl describe daemonset my-daemon

Check Pods
- kubectl get pods -o wide

Check Node eligibility
- labels
- taints
- affinity rules

## Key Takeaways

- DaemonSet runs one Pod per Node
- Automatically handles Node changes
- Used for infrastructure-level workloads
- Works with:
  - nodeSelector
  - affinity
  - tolerations
- Not used for scaling applications

## Useful Commands

List DaemonSets
- kubectl get daemonsets

Describe DaemonSet
- kubectl describe daemonset my-daemon

Delete DaemonSet
- kubectl delete daemonset my-daemon
