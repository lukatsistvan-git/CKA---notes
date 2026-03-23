# Service Accounts

## Overview

A **ServiceAccount (SA)** in Kubernetes is a special identity used by:
* Pods
* applications
* system components

It enables authentication to the **Kubernetes API server**.

Unlike user accounts:
* managed by Kubernetes
* namespace-scoped

---

## Default Behavior

Each namespace has a default ServiceAccount:
* name: `default`

If a Pod does not specify one:
* it automatically uses the `default` SA

---

## How ServiceAccounts Work

When a Pod uses a ServiceAccount:
* Kubernetes automatically injects credentials into the Pod

Mounted as a projected volume:
/var/run/secrets/kubernetes.io/serviceaccount/

Contains:

* token → API authentication
* ca.crt → cluster CA certificate
* namespace → current namespace

---

## Example: Using ServiceAccount in Pod

spec:
  serviceAccountName: my-sa

If not specified:
* default ServiceAccount is used

---

## Token Changes (Kubernetes 1.24+)

### Old Behavior

* Tokens stored as Secrets
* Long-lived
* Automatically created

---

### New Behavior (Recommended)

* Tokens are **short-lived**
* Generated dynamically using **TokenRequest API**
* Mounted as **projected volumes**

---

## RBAC and ServiceAccounts

ServiceAccounts have **no permissions by default** (beyond minimal discovery permissions).

Access is granted via:
* Role + RoleBinding
* ClusterRole + ClusterRoleBinding

---

## Example: RoleBinding for ServiceAccount

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: monitoring
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io

---

## Common Use Cases

### Pods accessing API

* applications call Kubernetes API securely

---

### CI/CD Systems

* Jenkins, GitLab, ArgoCD
* scoped permissions via SA

---

### Monitoring Tools

* Prometheus
* reads metrics from cluster

---

### External Access

* token can be used outside cluster
* authenticate via API server

---

## Disabling Automatic Token Mount

For security:
spec:
  automountServiceAccountToken: false

---

## Important Notes

* ServiceAccounts are **namespace-scoped**
* Each Pod can use only one ServiceAccount
* Tokens are automatically rotated (new mechanism)

---

## Security Best Practices

* Do not use default ServiceAccount in production
* Create dedicated SA per application
* Apply least privilege via RBAC
* Disable token mounting if not needed
* Use short-lived tokens

---

## Common Pitfalls

* Forgetting to assign ServiceAccount to Pod
* Over-permissioning ServiceAccounts
* Assuming default SA is safe
* Not restricting RBAC permissions
* Exposing tokens unintentionally

---

## Key Takeaways

* ServiceAccount = identity for Pods and workloads
* Used for API authentication
* Integrated with RBAC for authorization
* Tokens are now short-lived and dynamic
* Critical for secure in-cluster communication

---

## Useful Commands

Create ServiceAccount:
- kubectl create serviceaccount NAME

List ServiceAccounts
- kubectl get serviceaccounts -n my-namespace

Describe ServiceAccount
- kubectl describe sa my-sa -n my-namespace

Generate Token Manually:
- kubectl create token my-serviceaccount -n my-namespace
