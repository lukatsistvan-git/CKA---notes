# Multi-Container Pods

## Overview

A multi-container Pod is a Pod that runs multiple containers together.

These containers:
- run on the same node
- share the same network namespace
- can share storage (volumes)

They are designed to work together as a single unit.

---

## Key Characteristics

### Shared Network

- All containers share the same IP address
- Communicate via:
  - localhost
  - same port space

### Shared Storage

- Containers can share data using volumes

### Same Lifecycle

- Containers are scheduled and started together
- If the Pod is deleted → all containers are terminated

---

## Basic Example

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
    - name: app
      image: simple-webapp
      ports:
        - containerPort: 8080
    - name: log-agent
      image: log-agent

---

## Design Patterns

### Sidecar Pattern

The most common pattern.
A sidecar container extends the main application.

Use Cases:
- log collection (e.g., Fluentd)
- configuration reload
- proxy (e.g., Envoy)
- monitoring agents

Example:
- App container → generates logs
- Sidecar → collects and forwards logs

### Adapter Pattern

Used to transform output of the main application.

Use Cases:
- convert logs or metrics into a standard format
- make legacy apps compatible with modern systems

Example:
- App → outputs custom metrics
- Adapter → converts to Prometheus format

### Ambassador Pattern

Acts as a proxy or gateway for external communication.

Use Cases:
- database proxy
- API gateway
- service mesh sidecar

Example:
- App → communicates with localhost
- Ambassador → forwards to external service

---

## Communication Between Containers

Containers in the same Pod:
- communicate via localhost
- can share files via volumes

Example:
- App writes logs to /var/log/app.log
- Sidecar reads the same file

---

## When to Use Multi-Container Pods

Use when containers are:
- tightly coupled
- need to share resources
- must run together

Do NOT use when:
- services are independent → use separate Pods

---

## Lifecycle Behavior

- All containers start together
- If one container crashes:
  - Pod may restart (depending on restartPolicy)
- Containers are not isolated from each other’s lifecycle

---

## Init Containers (Relation - covered in init-containers.md)

- Init containers run before main containers
- Not part of the main runtime
- Used for setup tasks

---

## Troubleshooting

One container failing, check logs individually:
- kubectl logs pod-name -c container-name

Identify containers:
- kubectl get pod pod-name -o jsonpath="{.spec.containers[*].name}"

Debug shared issues, check:
- volume mounts
- port conflicts
- resource limits

Common Mistakes:
- Putting unrelated services in one Pod
- Ignoring resource sharing (CPU/memory)
- Debugging only one container
- Not using sidecar when appropriate

---

## Key Takeaways

- Multi-container Pods run tightly coupled containers
- Share:
  - network (localhost)
  - storage (volumes)
- Common patterns:
  - Sidecar
  - Adapter
  - Ambassador
- Best used for supporting components, not independent services

---

## Useful Commands

Get Pod details:
- kubectl get pod pod-name -o yaml

Check logs per container:
- kubectl logs pod-name -c container-name

Describe Pod:
- kubectl describe pod pod-name
