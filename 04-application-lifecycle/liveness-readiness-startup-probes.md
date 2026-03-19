# Liveness, Readiness and Startup Probes

## Overview

Kubernetes provides probes to determine the health and state of containers.

These probes enable self-healing applications by:
- restarting unhealthy containers
- controlling traffic to Pods
- improving reliability

There are three main types:
- Liveness Probe
- Readiness Probe
- Startup Probe

---

## Liveness Probe

### Purpose

- Determines if a container is still running correctly.
- “Is the application alive?”

### Behavior

If the liveness probe fails:
- Kubernetes restarts the container

### When to Use

- Application can get stuck (deadlock, infinite loop)
- Process is running but not functioning correctly

### Example

livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 10

---

## Readiness Probe

### Purpose

- Determines if a container is ready to serve traffic.
- “Can the application handle requests?”

### Behavior

- If the readiness probe fails:
  - Pod is removed from Service endpoints
  - No traffic is sent to it
- Container is NOT restarted

### When to Use

- App needs time to initialize
- Depends on external services (DB, cache)
- Temporary overload handling

### Example

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5

---

## Startup Probe

### Purpose

- Used for slow-starting applications.

### Behavior

- Disables liveness and readiness checks until it succeeds
- Prevents premature restarts during startup

### Example

startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10

---

## Probe Types

Kubernetes supports three probe mechanisms:

### HTTP Probe

httpGet:
  path: /healthz
  port: 8080

### TCP Probe

tcpSocket:
  port: 3306

Checks if port is open.

### Exec Probe

exec:
  command:
    - cat
    - /tmp/healthy

Runs a command inside the container.

### Common Configuration Parameters

initialDelaySeconds: 10
periodSeconds: 5
timeoutSeconds: 2
failureThreshold: 3
successThreshold: 1

Meaning
- initialDelaySeconds → wait before first check
- periodSeconds → how often to check
- timeoutSeconds → probe timeout
- failureThreshold → failures before action
- successThreshold → successes to recover

---

## How Probes Work with Services

- Only Ready Pods receive traffic
- Readiness probe controls Service endpoints

---

## Troubleshooting

### Pod keeps restarting

Check:
- kubectl describe pod pod-name
- kubectl logs pod-name

Likely cause:
- failing liveness probe

### Pod not receiving traffic

Check:
- readiness probe status
- Service endpoints

Command:
- kubectl get endpoints

### Probe failing incorrectly

Common issues:
- wrong path or port
- app not ready yet
- timeout too low

---

## Best Practices

- Always define readiness probe
- Use liveness probe for long-running apps
- Use startup probe for slow apps
- Avoid overly aggressive probe settings
- Ensure endpoints are lightweight

---

## Key Takeaways

- Liveness → restarts unhealthy containers
- Readiness → controls traffic flow
- Startup → protects slow startup apps
- Probes are essential for self-healing systems

---

## Useful Commands

Describe Pod:
- kubectl describe pod pod-name

Check logs:
- kubectl logs pod-name

Check endpoints:
- kubectl get endpoints
