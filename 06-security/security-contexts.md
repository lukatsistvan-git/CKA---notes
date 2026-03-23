# Security Contexts

## Overview

A **SecurityContext** defines privilege and access control settings for:
* Pods
* Containers

It controls:
* user identity (UID/GID)
* privileges
* filesystem access
* Linux capabilities

---

## Container Root vs Host Root

Running as **root (UID 0)** inside a container:
* does NOT mean root access on the host
* container is isolated via namespaces and cgroups

However:
* still considered **high risk**
* should be avoided when possible

---

## Defining Security Context

SecurityContext can be defined at:
* Pod level
* Container level

---

## Example: Running as Non-Root

apiVersion: v1
kind: Pod
metadata:
  name: non-root-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false

---

## Important Fields

### runAsUser

* UID used to run the container

### runAsGroup

* primary GID for the process

### runAsNonRoot

Enforces non-root execution:
runAsNonRoot: true

* Pod fails if container runs as root

### fsGroup

* applies to mounted volumes
* ensures group ownership for file access

### allowPrivilegeEscalation

* controls if process can gain more privileges
* should be set to false for security

---

## Precedence Rules

Priority order:
1. Container securityContext
2. Pod securityContext
3. Dockerfile USER

Example:
* Dockerfile → USER 1001
* Pod → runAsUser: 1003
* Container → runAsUser: 1002

Result:
* container runs as UID **1002**

---

## Linux Capabilities

Instead of full root access, use **capabilities**.

Example:

securityContext:
  capabilities:
    drop:
    - ALL
    add:
    - NET_BIND_SERVICE

### Explanation

* drop: ALL → removes all capabilities
* add → grants only required permissions

Example:

* NET_BIND_SERVICE → allows binding to ports <1024

---

## Privileged Containers

securityContext:
  privileged: true

Effects:
* full access to host resources
* bypasses container isolation

Use only when necessary:
* CNI plugins
* low-level system tools

---

## Additional Security Controls

### readOnlyRootFilesystem

securityContext:
  readOnlyRootFilesystem: true

* prevents writes to container root filesystem
* reduces attack surface

### runAsNonRoot (Best Practice)

securityContext:
  runAsNonRoot: true

* enforces non-root execution
* prevents accidental privilege escalation

### seccompProfile

Restricts system calls:

securityContext:
  seccompProfile:
    type: RuntimeDefault

### capabilities vs privileged

* Prefer capabilities over privileged
* granular vs full access

---

## Pod vs Container Scope

* Pod-level → default for all containers
* Container-level → overrides Pod settings

---

## Best Practices

* Run containers as non-root
* Use read-only root filesystem
* Drop all capabilities, add only
* Disable privilege escalation
* Avoid privileged containers
* Use seccomp profiles

---

## Common Pitfalls

* Running as root unnecessarily
* Using privileged: true
* Not setting runAsNonRoot
* Forgetting fsGroup for volume access
* Over-permissioned capabilities

---

## Key Takeaways

* SecurityContext controls runtime security
* Root in container ≠ root on host, but still risky
* Prefer least privilege model
* Container-level settings override Pod-level
* Avoid privileged containers whenever possible

---

## Useful Commands

- kubectl describe pod
- kubectl get pod -o yaml
