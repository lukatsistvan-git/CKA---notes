# Resource Requests and Limits

## Overview

Kubernetes allows you to define **resource requests and limits** for containers.

These control:
- how much CPU and memory a container **is guaranteed**
- how much it is **allowed to use at most**

With this:
- ensures Pods get minimum required resources
- prevents one container from consuming everything
- scheduler makes decisions based on requests

---

## Core Concepts

### 1. Requests

**Requests** define the minimum amount of resources guaranteed for a container.
- used by the **scheduler**
- determines which Node can run the Pod

Example:

resources:
  requests:
    cpu: "250m"
    memory: "128Mi"

### 2. Limits

Limits define the maximum resources a container can use.

If exceeded:
- CPU → throttling (container is slowed down)
- Memory → container is terminated (OOMKilled)

Example:

resources:
  limits:
    cpu: "500m"
    memory: "256Mi"

## Units

CPU:
- 1 CPU = 1 vCPU / core
- 500m = 0.5 CPU
- 250m = 0.25 CPU

Memory:
- Measured in bytes
- Common units:
  - Mi (Mebibyte) = 1,048,576 bytes
  - Gi (Gibibyte)

## How Scheduling Works

The scheduler uses requests only. Limits are NOT used during scheduling.

Flow:
- Check Node available resources
- Compare with Pod requests
- Schedule Pod on a suitable Node

## ResourceQuota

ResourceQuota is applied at the namespace level.

It limits total resource usage:
- CPU
- memory
- number of Pods

Example:

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi

## LimitRange

LimitRange sets default values for Pods/containers.

Used when:
- requests or limits are not specified

Example:

apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "250m"
      memory: "128Mi"
    type: Container

## Common Mistakes

- No limits defined: → container may consume all memory → crash other Pods
- Requests too high: → Pod stays in Pending
- Limits too low: → frequent throttling or OOMKilled

## Troubleshooting

Pod Pending?
- kubectl describe pod <pod-name>

Look for:
- insufficient CPU
- insufficient memory

Check resource usage:
- kubectl top pod

Check node capacity:
- kubectl describe node node1

## Key Takeaways

- Requests define minimum guaranteed resources
- Limits define maximum allowed resources
- Scheduler uses requests only
- CPU → throttling
- Memory → OOM kill
- ResourceQuota and LimitRange provide cluster-level control

## Useful Commands

Show resource usage
- kubectl top pod
