# Taints and Tolerations

## Overview

**Taints and tolerations** control which Pods can be scheduled on which Nodes.

They work together to **restrict Pod placement**.
- **Taint** → applied to a Node → repels Pods
- **Toleration** → applied to a Pod → allows it to tolerate the taint

---

## Core Concept

Taints prevent Pods from being scheduled on a Node unless the Pod has a matching toleration.

Important:
- Taints do NOT guarantee that a Pod will be scheduled on a Node.
- They only prevent scheduling on certain Nodes.

---

## How It Works

Flow:
- Node has taint → Pod does not tolerate → Pod is NOT scheduled
- Node has taint → Pod tolerates → Pod CAN be scheduled

## Taints

Taints are applied to Nodes.

Format:
- key=value:effect

Example:
- kubectl taint nodes node1 key=value:NoSchedule

### Taint Effects

The effect defines how strictly the taint is enforced.

NoSchedule (most commonly used):
- Pods will NOT be scheduled on the Node
- unless they have a matching toleration

PreferNoSchedule:
- Kubernetes tries to avoid placing Pods on the Node
- but it is NOT strictly enforced

NoExecute:
- Pods without matching toleration are:
  - NOT scheduled
  - AND removed from the Node if already running

### Managing Taints

Add a taint:
- kubectl taint nodes node1 key=value:NoSchedule
Remove a taint:
- kubectl taint nodes node1 key:NoSchedule-
Modify a taint (there is no direct update):
- Remove existing taint
- Add new taint

## Tolerations

Tolerations are defined in Pod specifications.

Example:

tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"

### Toleration Operators

Equal (default):
- requires exact match of key and value

Example:

tolerations:
- key: "purpose"
  operator: "Equal"
  value: "database"
  effect: "NoSchedule"

Exists:
- only checks if the key exists
- ignores the value, should NOT be specified

Example:

tolerations:
- key: "purpose"
  operator: "Exists"
  effect: "NoSchedule"

Note:
- there are other type of opertators, like Gt, Lt.

## Troubleshooting
Pod not scheduled?

Check:
- kubectl describe pod <pod-name>

Look for:
- taint-related messages

Check node taints
- kubectl describe node node1

List nodes with taints
- kubectl get nodes -o yaml | grep -i taints -A 5

## Key Takeaways

- Taints repel Pods from Nodes
- Tolerations allow Pods to tolerate taints
- They control where Pods should NOT run
- They are often used with:
  - dedicated nodes
  - special workloads
- They work together with the scheduler

## Useful Commands

Add taint:
- kubectl taint nodes node1 key=value:NoSchedule

Remove taint:
- kubectl taint nodes node1 key:NoSchedule-

Describe node:
- kubectl describe node node1
