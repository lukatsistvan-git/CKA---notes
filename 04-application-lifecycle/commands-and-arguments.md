# Commands and Arguments

## Overview

In Kubernetes, you can control how a container starts using:
- command
- args

These fields allow you to override the default behavior defined in the container image.

---

## How It Works in Docker

A Docker container runs a main process.
If the process exits → the container stops

Example:
- docker run ubuntu sleep 5

This:
- starts a container
- runs sleep 5
- exits after 5 seconds

### ENTRYPOINT and CMD

In Docker:
- ENTRYPOINT → defines the executable
- CMD → provides default arguments

Example:
  ENTRYPOINT ["sleep"]
  CMD ["5"]

Result:
- docker run image → sleep 5
- docker run image 10 → sleep 10

To override the command completely:
- docker run --entrypoint sleep2.0 image 10

---

## Mapping to Kubernetes

Kubernetes uses different field names:
- command → equivalent to ENTRYPOINT
- args → equivalent to CMD

Important:
- command does NOT append to ENTRYPOINT
- It replaces it entirely

---

## Pod Definition Example

Using command and args separately:

apiVersion: v1
kind: Pod
metadata:
  name: sleeper
spec:
  containers:
    - name: sleeper-container
      image: ubuntu
      command: ["sleep"]
      args: ["10"]

Using only command:

apiVersion: v1
kind: Pod
metadata:
  name: sleeper
spec:
  containers:
    - name: sleeper-container
      image: ubuntu
      command:
        - sleep
        - "10"

---

## Override Rules

### Default Behavior, if you do not specify anything.
Kubernetes uses:
- image ENTRYPOINT
- image CMD

### Only args defined
args: ["10"]

- Overrides CMD only
- Keeps the original ENTRYPOINT

### command defined
command: ["sleep"]

- Overrides ENTRYPOINT completely
- Ignores the image ENTRYPOINT
- Uses provided command

### command + args
command: ["sleep"]
args: ["10"]

Fully overrides both:
- ENTRYPOINT
- CMD

---

## Common Use Cases

- Run debug commands (e.g., sleep, bash)
- Override application startup behavior
- Inject different runtime parameters
- Run one-off jobs

---

## Troubleshooting Tips

### Container exits immediately

Possible cause:
- Main process finished execution

Example:
- Running sleep 5 → container stops after 5 seconds

### Wrong command/args

Symptoms:
- CrashLoopBackOff
- Container fails instantly

Steps:
- Check Pod:
  - kubectl describe pod pod-name
- Check logs:
  - kubectl logs pod-name
- Verify command/args in YAML

### Debugging with temporary command

Run a container interactively:
command: ["sleep"]
args: ["3600"]

This keeps the container alive for debugging.

---

## Key Takeaways

- command = ENTRYPOINT
- args = CMD
- They override image defaults
- command fully replaces ENTRYPOINT
- args only replaces CMD
- Container stops when main process exits

---

## Useful Commands

Check Pod definition:
- kubectl get pod pod-name -o yaml

Describe Pod:
- kubectl describe pod pod-name

Check logs:
- kubectl logs pod-name
