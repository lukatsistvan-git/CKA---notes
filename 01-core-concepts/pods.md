# Pods

## What is a Pod

A **Pod** is the smallest deployable unit in Kubernetes.

It represents a **single instance of an application** running in the cluster.

A Pod can contain:
- one container (most common case)
- multiple tightly coupled containers

Containers inside the same Pod:
- share the **same network namespace**
- communicate via **localhost**
- share **storage volumes**

Pods are **ephemeral by nature**. If a Pod fails, Kubernetes usually replaces it with a new one.

---

## Scaling

Scaling in Kubernetes is performed by **adding or removing Pods**.
- **Scaling up** → creating additional Pods running the same application
- **Scaling down** → deleting Pods

Scaling never means adding more containers to an existing Pod. Instead, Kubernetes creates **new Pod instances**.

In real environments Pods are rarely created directly. Instead they are managed by higher level controllers such as:
- Deployments
- ReplicaSets
- Jobs
- DaemonSets

---

## Multi-Container Pods

Although most Pods contain a single container, multiple containers can run inside a Pod. This pattern is used when containers must work **closely together**.

Typical examples:
- **Sidecar container** – logging or monitoring
- **Proxy container** – networking helper
- **Adapter container** – data transformation

Example:
- Main application container
- Logging sidecar container
Both run inside the **same Pod**.

---

## Pod Definition (YAML)

Kubernetes objects are defined using **YAML manifests**.

A basic Pod definition contains the following sections:
- apiVersion – API version used by Kubernetes
- kind – the resource type
- metadata – name, labels, annotations
- spec – desired configuration

Example:
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx

---

## kubectl commands:

Create a Pod from a YAML file:
- kubectl create -f pod-definition.yaml
or
- kubectl apply -f pod-definition.yaml
Difference:
- create → creates a new resource
- apply → creates or updates the resource (recommended)

List all Pods in the current namespace:
- kubectl get pods
More detailed output:
- kubectl get pods -o wide

Get detailed information about a Pod:
- kubectl describe pod myapp-pod
Useful for troubleshooting events, container status, scheduling problems.

Deleting a Pod:
- kubectl delete pod myapp-pod

View container logs:
- kubectl logs myapp-pod
If the Pod contains multiple containers:
- kubectl logs myapp-pod -c nginx-container

Execute a command inside a container:
- kubectl exec -it <pod-name> -- /bin/sh
