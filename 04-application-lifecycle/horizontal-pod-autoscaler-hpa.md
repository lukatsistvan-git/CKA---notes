# Horizontal Pod Autoscaler (HPA)

## Overview

The Horizontal Pod Autoscaler (HPA) is a Kubernetes component that automatically adjusts the number of Pods in a workload based on resource usage.
This is called horizontal scaling.

HPA helps:
- maintain performance under load
- optimize resource usage
- reduce manual intervention

---

## How It Works

The HPA continuously monitors resource usage of Pods (e.g., CPU, memory).

It then:
- compares current usage to a defined target
- increases or decreases the number of Pods
- keeps the number of replicas between minReplicas and maxReplicas

HPA works with:
- Deployment
- ReplicaSet
- StatefulSet

---

## Scaling Logic

HPA uses this concept:
- If current usage > target → scale up
- If current usage < target → scale down
Scaling is based on average utilization across Pods.

---

## Metrics Source – Metrics Server

HPA requires metrics to function.

These are provided by:
- Metrics Server

The Metrics Server
- Collects CPU and memory usage from Pods
- Exposes metrics to Kubernetes API

Notes
- Without Metrics Server:
  - HPA does NOT work
  - kubectl top does NOT work
 
---

## Requirements

For HPA to work correctly:

1. Metrics Server must be installed

Check:
- kubectl get deployment metrics-server -n kube-system

2. Resource Requests must be defined

Example:

resources:
  requests:
    cpu: "100m"

Without this HPA cannot calculate utilization %.

---

## Example (YAML)

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 6
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60

Key Fields
- scaleTargetRef → target workload
- minReplicas → minimum Pods
- maxReplicas → maximum Pods
- metrics → defines scaling criteria

---

## Common Issues

### HPA not scaling

Check:
- kubectl describe hpa

### No metrics available

Cause:
- Metrics Server missing or not working

### CPU utilization = 0%

Cause:
- Missing resource requests in Pod spec

### Pods stuck in Pending

Cause:
- Not enough node capacity
Solution:
- scale cluster (add nodes)

---

## Best Practices

- Always define resource requests
- Set reasonable min/max replicas
- Avoid aggressive scaling thresholds
- Monitor behavior over time

---

## Key Takeaways

- HPA provides automatic horizontal scaling
- Based on resource usage
- Requires Metrics Server
- Works only if resource requests are defined
- Keeps replicas between min and max values

---

## Useful Commands

Create HPA:
- kubectl autoscale deployment myapp --cpu-percent=50 --min=1 --max=5

List HPA:
- kubectl get hpa

Describe HPA:
- kubectl describe hpa myapp

Check metrics:
- kubectl top pod
