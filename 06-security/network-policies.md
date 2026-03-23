# Network Policies

## Overview

A **NetworkPolicy** is a Kubernetes resource used to control network traffic between:
* Pods
* Pods and external endpoints

It allows defining:
* which Pods can communicate
* allowed traffic direction (Ingress / Egress)
* allowed ports and protocols

---

## Default Behavior

* By default, **all traffic is allowed** (no restrictions)

Once a NetworkPolicy selects a Pod:
* that Pod becomes **isolated**
* only explicitly allowed traffic is permitted

---

## Important Requirement

NetworkPolicies only work if the network plugin supports them:
* Calico
* Cilium
* Weave Net

Without support:
* policies are ignored

---

## Policy Types

### Ingress

* controls **incoming traffic** to Pods

### Egress

* controls **outgoing traffic** from Pods

### Both

You can define both:

policyTypes:
* Ingress
* Egress

---

## Basic Structure

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend

---

## Pod Selection

### podSelector (Target Pods)

Defines **which Pods the policy applies to**.

spec:
  podSelector:
    matchLabels:
      role: backend

Important:
* only selects Pods in the **same namespace**

---

## Traffic Sources and Destinations

### podSelector

* selects Pods in the same namespace

### namespaceSelector

* selects Pods from other namespaces

Example:

ingress:
- from:
  - namespaceSelector:
      matchLabels:
        name: production

### ipBlock

* allows traffic from specific IP ranges

Example:

* ipBlock:
    cidr: 10.0.0.0/24

---

## Combining Selectors

### namespaceSelector + podSelector

Both must match (**AND condition**):

ingress:
- from:
  - namespaceSelector:
      matchLabels:
        env: prod
    podSelector:
      matchLabels:
        role: frontend

### OR (multiple entries)

ingress:
- from:
  - podSelector:
      matchLabels:
        role: frontend
  - podSelector:
      matchLabels:
        role: admin

→ frontend OR admin allowed

---

## Example: Secure Access Policy

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-secure-access
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    - podSelector:
        matchLabels:
          role: admin
    ports:
    - protocol: TCP
      port: 443
  - from:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 443

---

## Egress Example

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
    ports:
    - protocol: UDP
      port: 53

---

## Default Deny Policy

To block all traffic:

### Deny All Ingress

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---

### Deny All Egress

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress

---

## Important Notes

* Policies are **additive (union of all rules)**
* No explicit deny rules
* Applies only to selected Pods
* Must define both ingress and egress if needed

---

## Best Practices

* Start with **default deny**
* Explicitly allow required traffic
* Use labels consistently
* Separate frontend/backend traffic
* Restrict egress (often forgotten)

---

## Common Pitfalls

* Network plugin does not support policies
* Forgetting egress rules
* Misunderstanding namespace scope
* Missing labels
* Assuming deny rules exist

---

## Key Takeaways

* NetworkPolicy controls Pod-level traffic
* Default = allow all
* Once applied → default deny for selected Pods
* Works only with supported CNI
* Rules are additive (no deny)

---

## Useful Commands

* kubectl get networkpolicies
* kubectl describe networkpolicy <name>
* kubectl get pods --show-labels
