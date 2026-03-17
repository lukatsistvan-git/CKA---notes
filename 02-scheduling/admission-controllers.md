# Admission Controllers

## Request Flow in Kubernetes

Every request to the API server goes through 3 stages:

1. **Authentication**
   - Who are you?

2. **Authorization**
   - Are you allowed to do this?

3. **Admission Controllers**
   - Should this request be allowed or modified?

---

## What is an Admission Controller?

An **Admission Controller** is a component that:
- validates requests
- mutates (modifies) requests
- can reject requests
before they are persisted in etcd.

---

## Two Main Types

### 1. Mutating Admission Controllers

- Modify incoming requests
- Run **first**

Examples:
- add default values
- inject sidecar containers
- add labels/annotations

---

### 2. Validating Admission Controllers

- Only validate requests
- Cannot modify
- Run **after mutating controllers**

Examples:
- enforce resource limits
- block invalid configurations

---

## Execution Order

Request → Authentication → Authorization → Mutating → Validating → etcd

---

## Common Admission Controllers

NamespaceLifecycle:
- ensures namespace exists
- prevents creation in terminating namespaces
- protects system namespaces

LimitRanger:
- enforces resource limits
- sets default requests/limits

ResourceQuota:
- enforces namespace-level quotas

DefaultStorageClass:
- assigns default storage class if not specified

NodeRestriction:
- prevents kubelets from modifying unauthorized resources
- important for security

---

## Webhooks

MutatingAdmissionWebhook:
- external service can modify requests

ValidatingAdmissionWebhook:
- external service validates requests

---

## Why They Matter

- Security - block unsafe workloads
- Policy enforcement - enforce standards
- Automation - inject defaults
- Resource control - prevent overuse

---

## Enabling Admission Controllers

Configured in API server file:
- /etc/kubernetes/manifests/kube-apiserver.yaml

Example:
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota

---

## Important Notes

- Mutating controllers run before validating
- Order matters
- Not all controllers are enabled by default
- Changes require API server restart (handled automatically by kubelet)

---

## Troubleshooting

Request rejected?
- kubectl describe pod <pod-name>

Check:
- admission errors
- validation failures

Check API server config
- cat /etc/kubernetes/manifests/kube-apiserver.yaml

---

## Key Takeaways

- Admission Controllers are the final gate before object creation
- Two types:
  - Mutating (modify)
  - Validating (verify)
- Order is critical
- Essential for:
  - security
  - policy enforcement
  - automation
