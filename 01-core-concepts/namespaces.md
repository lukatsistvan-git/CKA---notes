# Namespaces

## What is a Namespace

A **Namespace** is a logical partition in Kubernetes that allows you to isolate resources within the same cluster.

Namespaces are used to:
- separate environments (dev, test, prod)
- isolate teams or projects
- organize resources
- apply resource limits and policies

---

## Why Namespaces are Important

In large clusters, multiple applications and teams share the same infrastructure.
Namespaces help:
- avoid naming conflicts
- control resource usage
- improve organization
- simplify access control (RBAC)

---

## Default Namespaces

Kubernetes comes with several built-in namespaces:
- default - Default namespace for user resources
- kube-system - Kubernetes system components
- kube-public - Public resources (readable by all users)
- kube-node-lease - Node heartbeat information

---

## Creating Resources in a Namespace

By default, resources are created in the **default** namespace.

To specify a namespace in a YAML file:
metadata:
  name: myapp-pod
  namespace: dev

---

## Creating a Namespace
Using kubectl:
- kubectl create namespace dev

Using YAML:
apiVersion: v1
kind: Namespace
metadata:
  name: dev

Apply / Create from YAML:
- kubectl create -f namespace.yaml
- kubectl apply -f namespace.yaml

---

## Working with Namespaces

List Pods in default namespace
- kubectl get pods

List Pods in a specific namespace
- kubectl get pods --namespace=kube-system
or
- kubectl get pods -n kube-system

List resources in all namespaces
- kubectl get pods --all-namespaces

---

## Switching Default Namespace

Instead of specifying the namespace in every command, you can change the current context:
- kubectl config set-context --current --namespace=dev

After this, all commands will use the dev namespace by default.

---

## Resource Quotas

Namespaces can have resource limits using ResourceQuota.
This controls how much CPU, memory, and how many objects can be used.

Example:
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi

This means:
- maximum 10 Pods
- total requested CPU ≤ 4
- total requested memory ≤ 5Gi
- total CPU limit ≤ 10
- total memory limit ≤ 10Gi

---

## How Namespaces Provide Isolation

Namespaces isolate:
- resource names (Pods, Services, Deployments)
- configurations
- access control (via RBAC)
- resource quotas

However, they do NOT fully isolate:
- network (unless NetworkPolicies are used)
- nodes
- cluster-wide resources

You can check the print the supported namespaced resources:
- kubectl api-resources --namespaced=true
or the supported non-namespaced resources:
- kubectl api-resources --namespaced=false

---

## Key Takeaways

- Namespaces provide logical isolation
- They help organize resources in a cluster
- They enable resource control and quotas
- They are essential for multi-environment setups
- DNS names (Fully Qualified Domain Name - FQDN) include the namespace

---

## Useful Commands

Create namespace
- kubectl create namespace dev

List namespaces
- kubectl get ns

Create resource in namespace
- kubectl apply -f pod.yaml -n dev

Delete namespace
- kubectl delete namespace dev
