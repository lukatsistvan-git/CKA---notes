# Node Affinity

## Overview

**Node affinity** allows you to control which Nodes a Pod can be scheduled on, based on Node labels.
It is a more expressive and flexible version of `nodeSelector`.

---

## Why Use Node Affinity?

- control workload placement
- use specific hardware (e.g. SSD, GPU)
- separate environments (prod, dev)
- enforce compliance or locality rules

---

## Node Selector vs Node Affinity

nodeSelector:
- it is a simple key:value pair
- limited flexibility

nodeAffinity
- more advanced
- multiple operators
- high flexibility

---

## Core Concept

Node affinity works by matching **Pod requirements** with **Node labels**.

Example Node label:
labels:
  disktype: ssd

## Types of Node Affinity

1. requiredDuringSchedulingIgnoredDuringExecution

Hard requirement:
- Pod will NOT be scheduled if conditions are not met

Example (Hard Requirement)
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
              - ssd
              
2. preferredDuringSchedulingIgnoredDuringExecution

Soft requirement:
- Scheduler prefers matching Nodes
- Pod can still be scheduled elsewhere

Example (Soft Preference)
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
              - ssd

## Operators (matchExpressions)

Node affinity supports multiple operators:
- In → value is in the list
- NotIn → value is NOT in the list
- Exists → key exists
- DoesNotExist → key does not exist
- Gt → greater than (numeric)
- Lt → less than (numeric)

Example (Multiple Conditions)
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
              - ssd
          - key: zone
            operator: NotIn
            values:
              - us-east-1b

This means:
- Node must have disktype=ssd
- Node must NOT be in us-east-1b

## How Scheduling Works

The scheduler:
- Filters Nodes based on required rules
- Scores Nodes based on preferred rules
- Selects the best matching Node

Important Notes:
- Node affinity is evaluated only during scheduling
- After the Pod is running, changes in Node labels are ignored (IgnoredDuringExecution)

## Key Takeaways

- Node affinity controls where Pods SHOULD run
- Supports complex matching rules
- Has:
  - hard constraints (required)
  - soft preferences (preferred)
- Works together with:
  - scheduler
  - taints and tolerations
