# Environment Variables

## Overview

In Kubernetes, environment variables can be used to configure applications running inside containers.
They are defined in the Pod (or Deployment) specification and injected into the container at runtime.

---

## Basic Syntax

Environment variables are defined using the env field:

apiVersion: v1
kind: Pod
metadata:
  name: env-example
spec:
  containers:
    - name: my-container
      image: busybox
      command: ["sh", "-c", "echo Hello $MY_VAR; sleep 3600"]
      env:
        - name: MY_VAR
          value: "HelloFromKubernetes"

Key Fields
- env → list of environment variables
- name → variable name
- value → variable value

---

## Using Multiple Environment Variables

env:
  - name: APP_ENV
    value: "production"
  - name: APP_DEBUG
    value: "false"

---

## Environment Variables from ConfigMap

You can load values from a ConfigMap:

env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: color

---

## Environment Variables from Secret

You can load sensitive data from a Secret:

env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password

---

## Using envFrom

Instead of defining variables one by one, you can import all values:

From ConfigMap
envFrom:
  - configMapRef:
      name: app-config

From Secret
envFrom:
  - secretRef:
      name: db-secret

This creates environment variables for each key in the resource.

---

## Using Downward API (Pod Metadata)

You can expose Pod metadata as environment variables:

env:
  - name: POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name

Other examples:
- metadata.namespace
- spec.nodeName
- status.podIP

---

## Default Behavior

Environment variables are set at container start time

They do NOT update dynamically if the source (ConfigMap/Secret) changes

---

## Common Use Cases

- Application configuration (env, mode, flags)
- Database connection details
- Feature flags
- Injecting runtime metadata (Pod name, namespace)

---

## Troubleshooting

### Variable not available

Check:
- YAML syntax
- Correct name and key
- Resource exists (ConfigMap/Secret)

### Value not updated

Explanation:
- Environment variables are static

Solution:
- Restart Pod (or rollout Deployment)

### Debugging

Check environment variables inside container:
- kubectl exec -it pod-name -- env

---

## Key Takeaways

- Environment variables are defined using env
- Can be set manually or from:
  - ConfigMap
  - Secret
  - Pod metadata (Downward API)
- envFrom allows bulk import
- Values are static after container start
- Commonly used for application configuration

---

## Useful Commands

Check Pod definition:
- kubectl get pod pod-name -o yaml

Exec into container:
- kubectl exec -it pod-name -- sh

Print environment variables:
- env
- printenv
