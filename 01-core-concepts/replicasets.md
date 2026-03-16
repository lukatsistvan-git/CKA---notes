# ReplicaSets

## What is a ReplicaSet

A **ReplicaSet** ensures that a specified number of **Pod replicas** are running at all times.
If a Pod crashes, is deleted or becomes unreachable the ReplicaSet **automatically creates a new Pod** to maintain the desired number of replicas.
ReplicaSets continuously monitor Pods that match their label selector.
If the number of matching Pods:
- drops below the desired count → new Pods are created
- exceeds the desired count → extra Pods are terminated

This provides:
- High availability
- Self-healing
- Horizontal scaling

ReplicaSets are usually **not created directly**. In most production environments they are managed by **Deployments**.

Key Takeaways:
- ReplicaSets maintain a stable number of running Pods
- They provide self-healing and scaling
- They rely on label selectors to identify Pods
- In real environments ReplicaSets are typically managed by Deployments

---

## ReplicationController vs ReplicaSet

The **ReplicationController** is the older replication mechanism in Kubernetes.
ReplicaSet is the **next generation replacement** with improved label selection capabilities.

---

## ReplicaSet Example

Example definition:

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      type: frontend
  template:
    metadata:
      labels:
        app: myapp
        type: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx

Important fields:
- replicas → desired number of Pods
- selector → how the ReplicaSet identifies the Pods it manages
- template → Pod template used to create new Pods

---

## Commands

Create ReplicaSet:
- kubectl create -f replicaset-definition.yaml
- kubectl apply -f replicaset-definition.yaml

Edit ReplicaSet:
- kubectl edit replicaset myapp-replicaset

List ReplicaSets:
- kubectl get rs

View detailed information:
- kubectl describe rs myapp-replicaset

Scale ReplicaSet:
- kubectl scale rs myapp-replicaset --replicas=6

Delete ReplicaSet:
- kubectl delete rs myapp-replicaset
