# Priority Classes

## Overview

A **PriorityClass** defines the importance of Pods in Kubernetes.

It is used by the scheduler to decide:
- which Pods to schedule first
- which Pods can preempt others when resources are limited

---

## Why It Matters

When the cluster does not have enough resources:
- higher priority Pods are scheduled first
- lower priority Pods may be **evicted (preempted)**

---

## Core Concept

Each Pod gets a **priority value** from its PriorityClass.
- higher value → higher priority
- lower value → lower priority

---

## Creating a PriorityClass

Example:

apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "High priority class for critical workloads"

- value:
  - Integer value
  - Range: -2147483648 to 2147483647
  - Higher value = higher priority

- preemptionPolicy
  - Defines if the Pod can preempt others:
  - PreemptLowerPriority (default)
  or
  - Never

- globalDefault
  - If true → becomes default for Pods without priority
  - Only one PriorityClass can have this

---

## Using PriorityClass in a Pod

spec:
  priorityClassName: high-priority

---

## Preemption

Preemption means:
- Kubernetes removes lower-priority Pods to make room for higher-priority ones.

Conditions:
- Node does not have enough resources
- Higher priority Pod is pending
- Preemption is allowed

---

## Scheduling Flow with Priority

- Pods are placed in a scheduling queue
- Higher priority Pods are processed first
- If no resources:
  - scheduler may trigger preemption
- Lower priority Pods are terminated

---

## Built-in Priority Classes

Kubernetes provides system-critical priorities:
- system-cluster-critical → very high priority
- system-node-critical → even higher priority

Used for:
- core cluster components
- system stability

---

## Important Notes

- PriorityClass does NOT reserve resources
- It only affects scheduling order and preemption
- Works together with:
  - resource requests
  - scheduler decisions

---

## Example Scenario

Cluster is full:
- Pod A → priority 1000
- Pod B → priority 100000

Result:
- Pod B preempts Pod A
- Pod A is terminated

---

## Common Mistakes

- Assuming priority guarantees scheduling
  - It does NOT
  - only increases chances

- No resource requests
  - scheduler cannot make proper decisions

- Overusing high priority
  - can destabilize cluster
 
---

## Troubleshooting

Pod stuck in Pending.
- kubectl describe pod <pod-name>

Look for:
- insufficient resources
- preemption messages

Check PriorityClasses
- kubectl get priorityclass

Describe PriorityClass
- kubectl describe priorityclass high-priority

---

## Useful Commands

Create
- kubectl apply -f priorityclass.yaml

List
- kubectl get priorityclass

Describe
- kubectl describe priorityclass high-priority

Delete
- kubectl delete priorityclass high-priority

---

## Key Takeaways

- PriorityClass defines Pod importance
- Higher priority Pods are scheduled first
- Enables preemption
- Does NOT guarantee resources
- Critical for:
  - production workloads
  - system stability
