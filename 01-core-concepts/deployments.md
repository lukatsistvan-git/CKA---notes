# Deployments

## What is a Deployment

A **Deployment** is a Kubernetes object used to manage the lifecycle of applications.

It allows you to:
- deploy applications
- scale Pods
- update container versions
- roll back to previous versions
- pause and resume updates

Deployments manage **ReplicaSets**, which in turn manage **Pods**.

Architecture: Deployment → ReplicaSet → Pods
Because of this, Pods are usually **not created directly in production environments**.

---

## Why Deployments are Important

Deployments provide several key capabilities:
- **Declarative updates**
- **Rolling updates**
- **Rollbacks**
- **Scaling**
- **Self-healing**

They ensure that the **desired state of the application** is always maintained.

---

## Deployment Definition Example

Example YAML definition:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx-container
        image: nginx

Important fields:
- replicas - desired number of Pods
- selector - identifies Pods managed by the Deployment
- template - Pod definition used to create Pods

---

## Updating a Deployment

- update the YAML file and apply changes
- use set image command (This updates the running Deployment, but does not modify the YAML file.)

---

## Rollouts and Revisions

When a Deployment is first created, Kubernetes performs a rollout.
Each rollout creates a Deployment Revision.
Revisions track changes such as container image updates or configuration updates.
This allows Kubernetes to roll back to previous versions.

---

## Rollback

If a deployment update causes problems, it can be rolled back with rollout undo command.
This restores the previous Deployment revision.

---

## Deployment Strategies

Kubernetes supports two deployment strategies.

1. Recreate Strategy

All old Pods are terminated before new ones are created.

Process:
- old Pods → terminated  
- new Pods → created

Result:
- temporary downtime
- application becomes unavailable during update

2. Rolling Update Strategy (Default)

Pods are replaced gradually.

Process: (repeated until all Pods are updated)
- old Pod removed  
- new Pod created

Benefits:
- zero downtime
- smooth updates
- continuous availability

---

## Key Takeaways

Deployments manage ReplicaSets
ReplicaSets manage Pods
Deployments enable rolling updates
Deployment revisions allow rollbacks
Rolling updates provide zero downtime deployments

---

## Useful Commands

Create Deployment:
- kubectl create -f deployment-definition.yaml
- kubectl apply -f deployment-definition.yaml
or
- kubectl create deployment my-deployment --image nginx --replicas 3

List Deployments:
- kubectl get deployments

Checking Deployment Details:
- kubectl describe deployment myapp-deployment

Update image:
- kubectl set image deployment/myapp-deployment nginx-container=nginx:1.25

Check rollout status:
- kubectl rollout status deployment/myapp-deployment

View rollout history:
- kubectl rollout history deployment/myapp-deployment

Rollback:
- kubectl rollout undo deployment/myapp-deployment
