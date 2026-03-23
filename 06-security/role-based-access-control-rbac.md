#  Role-Based Access Control (RBAC)

## Overview

Kubernetes uses **authorization** to determine what an authenticated user can do.

Authorization happens **after authentication** and is handled by the **kube-apiserver**.

---

## Authorization Modes

Kubernetes supports multiple authorization modes:

### AlwaysAllow

* Every request is allowed
* Used only for testing
* Not secure

---

### AlwaysDeny

* Every request is denied
* Used for debugging

---

### Node

* Handles permissions for kubelets
* Allows node to:
  * read Pods assigned to it
  * report node status

---

### ABAC (Attribute-Based Access Control)

* Policy defined in JSON file
* Static and hard to manage
* **Deprecated / not recommended**

---

### RBAC (Role-Based Access Control)

* Most commonly used and recommended
* Uses Kubernetes objects:
  * Role
  * RoleBinding
  * ClusterRole
  * ClusterRoleBinding

---

### Webhook

* Delegates authorization to external service
* Useful for enterprise integrations

---

## Authorization Mode Configuration

Set via kube-apiserver:
--authorization-mode=Node,RBAC,Webhook

### How It Works

* Modes are evaluated in order
* Each mode returns:
  * allow → request accepted
  * deny → request rejected immediately
  * no opinion → next mode

Final result:
* At least one **allow**
* No **deny**

---

## RBAC Overview

RBAC controls access by defining:

* **who** (user, group, service account)
* **what** (resources)
* **which actions** (verbs)

---

## RBAC Core Objects

### Role

Defines permissions **within a namespace**.

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: my-namespace
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

Fields Explained:
- apiGroups: "" → core API group (pods, services...)
- resources: Kubernetes resource types, like, pods, deployments...
- verbs: (get, list, watch, create, update, delete)

---

### ClusterRole

Can grant access to:
- cluster-scoped resources (like nodes)
- non-resource endpoints (like /healthz)
- namespaced resources (like Pods), across all namespaces
- to allow a particular user to run kubectl get pods --all-namespaces

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]

### RoleBinding

Assigns a Role to users or service accounts.

apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io

---

### ClusterRoleBinding

apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io

---

## Resource-Level Permissions

You can restrict access to specific objects.

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: single-pod-reader
  namespace: my-namespace
rules:
- apiGroups: [""]
  resources: ["pods"]
  resourceNames: ["nginx-pod"]
  verbs: ["get", "watch"]

---

## Namespace Scope

* Role → namespace scoped
* RoleBinding → namespace scoped
* ClusterRole → cluster-wide
* ClusterRoleBinding → cluster-wide

---

## Important Notes

* RBAC works on **API requests**, not directly on objects
* Permissions are **additive** (no deny rules in RBAC)
* Evaluation is based on all applicable bindings

---

## Best Practices

* Follow **least privilege principle**
* Use Roles instead of ClusterRoles when possible
* Avoid using cluster-admin unnecessarily
* Use ServiceAccounts for applications
* Regularly audit permissions

---

## Common Pitfalls

* Forgetting namespace in RoleBinding
* Using Role instead of ClusterRole (or vice versa)
* Missing apiGroup
* Assuming RBAC supports explicit deny (it does not)
* Over-permissioning users

---

## Key Takeaways

* RBAC is the default authorization mechanism
* Role + RoleBinding = namespace access
* ClusterRole + ClusterRoleBinding = cluster access
* No deny rules → only allow
* kubectl auth can-i is essential for debugging

---

## Useful Commands

Quick Creation Commands

Create Role:
- kubectl create role pod-reader --verb=get,list --resource=pods -n my-namespace

Create RoleBinding:
- kubectl create rolebinding jane-read-pods --role=pod-reader --user=jane -n my-namespace

View Roles
- kubectl get roles -n my-namespace
- kubectl describe role pod-reader -n my-namespace

View RoleBindings
- kubectl get rolebindings -n my-namespace
- kubectl describe rolebinding read-pods-binding -n my-namespace

Cluster-wide Resources
- kubectl get clusterroles
- kubectl get clusterrolebindings

Checking Permissions
- kubectl auth can-i
Examples
- kubectl auth can-i get pods --as=jane -n my-namespace
- kubectl auth can-i create deployments --as=john -n dev
- kubectl auth can-i delete pods --as=system:serviceaccount:default:my-serviceaccount
