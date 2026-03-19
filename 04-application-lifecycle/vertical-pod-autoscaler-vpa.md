# Vertical Pod Autoscaler (VPA)

## Overview

The Vertical Pod Autoscaler (VPA) is a Kubernetes component that automatically adjusts the CPU and memory requests (and optionally limits) of Pods based on actual usage.

It helps:
- optimize resource allocation
- reduce overprovisioning and underprovisioning
- minimize manual tuning

The VPA continuously:
- observes resource usage of Pods
- generates recommendations for CPU and memory
- optionally applies those recommendations

The behavior depends on the configured update mode.

---

## Update Modes

The VPA supports multiple update strategies:

### Off

- Only provides recommendations
- Does NOT modify Pods

### Initial

- Applies recommendations only at Pod creation time
- Existing Pods are NOT modified

### Recreate

- Deletes Pods and recreates them with new resource values
- Causes temporary disruption

### Auto

- Fully automatic mode
- VPA decides how to apply updates:
  - usually via Recreate
  - may use in-place update if supported

### In-Place Scaling

- Controlled by feature gate: InPlacePodVerticalScaling
- Still not universally stable across all environments
- Without it → Pods must be recreated

---

## VPA Components

The VPA consists of three main components:

### VPA Recommender

- Monitors CPU and memory usage (via metrics)
- Calculates recommended:
  - requests
  - optionally limits
- Stores recommendations in the VPA object

View recommendations:
- kubectl describe vpa myapp-vpa

### VPA Updater

- Compares running Pods with recommended values
- If difference is significant and updateMode allows:
  - Deletes Pods → new ones will be created with updated resources
  - OR performs in-place update (if enabled)

Important
- Updater does NOT directly modify Pod spec
- It triggers recreation or update
- Actual injection happens during admission phase

### VPA Admission Controller

- Intercepts Pod creation requests
- Injects recommended resource values into:

resources:
  requests:

- Works only if updateMode ≠ Off

---

## Example YAML
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"

---

## Requirements

- Metrics provider (commonly Metrics Server)
- VPA components must be installed manually:
  - not part of core Kubernetes by default

---

## Notes

### Resource Requests vs Limits

- VPA primarily focuses on requests
- Limits may also be adjusted depending on configuration

### Interaction with HPA

- Using HPA and VPA together is complex
- Rule:
  - Avoid using both on the same resource metric
  - Example:
    - HPA → CPU, VPA → memory
    - Both controlling CPU → conflict

### Pod Disruption

- In Recreate or Auto mode:
- Pods may be restarted
- Can cause downtime if not handled properly

### Single Replica Case

- If a workload has only 1 replica:
  - VPA may NOT evict the Pod immediately
  - To avoid downtime
- However:
  - behavior depends on policies and disruption settings

### Limitations

- May cause restarts (disruptive)
- Not ideal for latency-sensitive apps without redundancy
- Requires careful tuning in production

---

## When to Use VPA

- Long-running workloads
- Unknown or variable resource usage
- Cost optimization scenarios

---

## Key Takeaways

- VPA adjusts Pod resource requests dynamically
- Works based on observed usage
- Has multiple update strategies
- Uses 3 components:
  - Recommender
  - Updater
  - Admission Controller
- May restart Pods depending on mode
- Not enabled by default

---

## Useful Commands

Create VPA:
- kubectl apply -f vpa.yaml

Describe VPA:
- kubectl describe vpa myapp-vpa
