# Image Security

## Overview

Container image security is critical in Kubernetes because:
* images define what runs in your cluster
* compromised images can lead to security breaches
* affects application integrity and reliability

---

## Default Image Registry

If you define an image like:
image: nginx

Kubernetes interprets it as:
docker.io/library/nginx:latest

### Components

* Registry: docker.io
* Namespace: library (official images)
* Image: nginx
* Tag: latest (default if not specified)

---

## Best Practice: Explicit Image Reference

Always specify:
* full registry path
* explicit tag

Example:
image: docker.io/library/nginx:1.25

Why:
* avoids unexpected updates
* ensures reproducibility

---

## Image Pull Policy

Controls when Kubernetes pulls images.

Values:
* Always → always pull image
* IfNotPresent → pull only if not cached
* Never → never pull

Default behavior:
* latest tag → Always
* other tags → IfNotPresent

---

## Private Image Registries

To use private images:
image: myregistry.example.com/myteam/app:1.0.0

You must authenticate using a **Secret**.

---

## Create Registry Secret

kubectl create secret docker-registry regcred 
--docker-server=myregistry.example.com 
--docker-username=myuser 
--docker-password=mypassword 
--docker-email=[myuser@example.com](mailto:myuser@example.com)

---

## Use imagePullSecrets in Pod

apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myregistry.example.com/myteam/app:1.0.0
    imagePullSecrets:
    - name: regcred

---

## Using imagePullSecrets with ServiceAccount

Instead of adding to every Pod:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-sa
imagePullSecrets:
- name: regcred

Then reference SA in Pod:
spec:
  serviceAccountName: my-sa

---

## Image Security Best Practices

### Use Trusted Images

* official images (Docker Hub library)
* verified publishers
* internal registries

---

### Avoid latest Tag

* not deterministic
* can introduce breaking changes

---

### Scan Images

Use tools to detect vulnerabilities:
* Trivy
* Clair
* Anchore

---

### Use Minimal Base Images

* alpine
* distroless

Reduces:
* attack surface
* vulnerabilities

---

### Enable Image Signing (Advanced)

* ensure image integrity
* tools:
  * cosign
  * Notary

---

### Restrict Image Sources

Use policies to allow only trusted registries:
* admission controllers
* OPA / Gatekeeper
* Pod Security / policies

---

## ImagePullBackOff Error

Common issue when pulling images.

### Causes

* wrong image name
* missing credentials
* private registry without auth
* network issues

### Debug

- kubectl describe pod <pod-name>

Look for:
* Events section
* image pull errors

---

## Important Notes

* Nodes pull images, not the API server
* imagePullSecrets must exist in the same namespace
* credentials are stored in Secrets

---

## Common Pitfalls

* forgetting imagePullSecrets
* using latest tag in production
* incorrect registry URL
* expired credentials
* assuming public access

---

## Key Takeaways

* Always use explicit image references
* Secure access to private registries
* Use imagePullSecrets or ServiceAccounts
* Scan and verify images
* Avoid latest tag in production

---

## Useful Commands

* kubectl create secret docker-registry regcred

* kubectl describe pod <pod-name>

* kubectl get events

kubectl get secrets
