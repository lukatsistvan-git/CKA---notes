# Init Containers

## Overview

An Init Container is a special type of container that runs before the main application containers in a Pod.
Its purpose is to perform initialization tasks required for the application to start properly.

---

## Common Use Cases

Init containers are typically used for:
- waiting for external services (e.g., database, API)
- downloading configuration files
- cloning a Git repository
- setting up permissions
- preparing shared volumes

---

## How It Works

- Init containers run before regular containers
- If multiple init containers exist:
  - they run sequentially (one by one)
- Each must complete successfully
- Only after that → main containers start

---

## Execution Flow

- Init container #1 starts
- If successful → Init container #2 starts
- After all init containers succeed → main containers start

---

## Failure Behavior

- If an init container fails:
  - Kubernetes restarts the Pod
- It keeps retrying until success

Result:
- The application never starts until initialization is complete

---

## Init Containers as Sidecars

After Kubernetes v1.29+, init containers can be used to implement sidecar containers.

If an init container is defined with:
- restartPolicy: Always
then:
- it starts during the init phase
- it does NOT terminate
- it continues running for the entire Pod lifecycle
so Kubernetes treats it as a sidecar container

---

## Example

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp
      image: busybox
      command: ['sh', '-c', 'echo App running! && sleep 3600']
  initContainers:
    - name: init-wait-db
      image: busybox
      command:
        - sh
        - -c
        - until nslookup my-database; do echo Waiting for DB; sleep 2; done;

What Happens in This Example
- init-wait-db runs first
- It repeatedly checks if my-database is reachable
- Only when successful:
  - the main container (myapp) starts
 
---

## Shared Resources

Init containers can share:
- Volumes → prepare files for main containers
- Network → access services (e.g., DB, API)

Example:
- Init container writes config to /data
- App container reads it from the same volume

---

## Troubleshooting

Pod stuck in Init phase

Check status:
- kubectl get pods

Output may show:
- Init:0/1
- Init:CrashLoopBackOff

Check init container logs
- kubectl logs pod-name -c init-container-name

Describe Pod
- kubectl describe pod pod-name

Look for:
- events
- errors
- retry messages

---

## Common Issues

- Infinite wait loops (e.g., service never becomes available)
- DNS resolution problems
- Wrong service name
- Network issues

---

## Best Practices

- Keep init containers simple and fast
- Avoid long-running logic
- Use timeouts when waiting for services
- Use them for setup only, not business logic

---

## Key Takeaways

- Init containers run before application containers
- They execute sequentially
- Must complete successfully before app starts
- Ideal for setup and preparation tasks
- Critical for reliable startup behavior
- Sidecar behavior with RestartPolicy: Always

---

## Useful Commands

Check Pod status:
- kubectl get pods

Check init container logs:
- kubectl logs pod-name -c init-container-name

Describe Pod:
- kubectl describe pod pod-name
