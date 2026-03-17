# Labels, Selectors and Annotations

## Labels

**Labels** are key-value pairs attached to Kubernetes objects.

They are used to:
- organize resources
- group related objects
- enable selection and filtering

Labels are widely used by:
- ReplicaSets
- Deployments
- Services
- Scheduler

Example:

metadata:
  labels:
    app: frontend
    env: prod

Labels allow Kubernetes to identify and group resources logically.
- all frontend Pods → app=frontend
- all production resources → env=prod

---

## Selectors

Selectors are used to select Kubernetes objects based on their labels.

They are used by:
- Services (to route traffic)
- ReplicaSets (to manage Pods)
- Deployments (via ReplicaSets)

---

### Types of Selectors
1. Equality-Based Selectors

Match exact key-value pairs.

Examples:
- app: frontend
- env: prod

Equivalent CLI usage:
- kubectl get pods -l app=frontend

2. Set-Based Selectors

More advanced selection using sets.

Examples:
- env in (dev, staging)
- env notin (prod)
- tier in (frontend, backend)

Selector Example (Service)
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80

This Service will route traffic to all Pods with:
- app: frontend

## How Labels and Selectors Work Together

Flow:
- Labels → applied to Pods
- Selectors → used to find matching Pods

Example:

Pod:
  labels:
    app: frontend

Service:
  selector:
    app: frontend

Result:
→ Service routes traffic to the Pod

## Annotations

Annotations are also key-value pairs, but they are NOT used for selection.
They are used to store additional metadata.

Annotation example
metadata:
  annotations:
    prometheus.io/scrape: "true"
    description: "Monitoring enabled"

### When to Use Annotations?

Annotations are used for:
- tooling (monitoring, logging)
- storing configuration data
- tracking changes

Example:
- kubectl.kubernetes.io/last-applied-configuration

## Key Takeaways

- Labels are used to identify and group resources
- Selectors are used to find resources based on labels
- Annotations store additional metadata
- Labels are critical for:
  - Services
  - ReplicaSets
  - Deployments

## Useful Commands

Show labels:
- kubectl get pods --show-labels

Filter by label:
- kubectl get pods -l app=frontend

Add label:
- kubectl label pod my-pod env=prod

Remove label:
- kubectl label pod my-pod env-
