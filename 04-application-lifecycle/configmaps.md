# ConfigMaps

## Overview

A ConfigMap is a Kubernetes resource used to store non-sensitive configuration data as key-value pairs.

Typical use cases:
- application configuration
- environment variables
- command-line arguments
- configuration files

---

## Why Use ConfigMaps

Instead of hardcoding configuration inside Pod definitions:
- separate configuration from application code
- reuse configuration across multiple Pods
- update configuration without rebuilding images
- improve maintainability

---

## Creating a ConfigMap

### From Command Line

kubectl create configmap my-config \
  --from-literal=ENV=production \
  --from-literal=PORT=8080

### From YAML

apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  ENV: production
  PORT: "8080"

### Apply:

kubectl apply -f my-config.yaml

### From File

kubectl create configmap my-config --from-file=config.properties

This stores file contents inside the ConfigMap.

---

## Using ConfigMaps in Pods

### As Environment Variables (envFrom)

envFrom:
  - configMapRef:
      name: my-config

Imports all keys as environment variables

### As Individual Environment Variables

env:
  - name: ENV
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: ENV

More control over specific variables

### As Files (Volume Mount)

volumes:
  - name: config-volume
    configMap:
      name: my-config
containers:
  - name: app
    image: my-app
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config

Result:
- Each key becomes a file
- Example:
  - /etc/config/ENV → contains "production"
  - /etc/config/PORT → contains "8080"
 
---

## How Updates Work

### Environment Variables

- Do NOT update automatically
- Require Pod restart (or Deployment rollout)

### Volume Mounts

- Updated automatically (eventually consistent)
- Kubelet refreshes files periodically

Important:
- Applications must be able to reload config dynamically

---

## Common Use Cases

- Application configuration (ENV, PORT, URLs)
- External service endpoints
- Feature flags
- Config files (JSON, YAML, properties)

---

## Limitations

- Not suitable for sensitive data (use Secrets instead)
- Size limit (~1MB)
- No encryption by default

---

## Troubleshooting

### ConfigMap not found

- kubectl get configmap
- Check name and namespace

### Wrong values

- kubectl describe configmap my-config

### Environment variables not updated

Cause:
- Env vars are static

Solution:
- Restart Pod / rollout Deployment

### Volume not mounted

Check:
- volume name matches
- volumeMount name matches
- mountPath exists

---

## Key Takeaways

- ConfigMaps store non-sensitive configuration
- Can be consumed as:
  - environment variables
  - files (volumes)
- envFrom imports all keys
- Changes to env vars require restart
- Volume-based configs can update dynamically

---

## Useful Commands

Create ConfigMaps:
- kubectl create configmap my-config --from-literal=ENV=production --from-literal=PORT=8080
- kubectl create configmap my-config --from-file=config.properties
- kubectl apply -f my-config.yaml

List ConfigMaps:
- kubectl get configmaps
- kubectl get cm

Describe ConfigMap:
- kubectl describe configmap my-config

View YAML:
- kubectl get configmap my-config -o yaml

Delete:
- kubectl delete configmap my-config
