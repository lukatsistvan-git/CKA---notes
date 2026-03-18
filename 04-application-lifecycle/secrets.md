# Secrets

## Overview

A Secret is a Kubernetes resource used to store sensitive data such as:
- passwords
- API tokens
- SSH keys
- TLS certificates

Secrets help separate sensitive data from:
- application code
- container images
- Pod definitions

---

## Why Use Secrets

Instead of hardcoding sensitive data:
- improves security
- enables centralized management
- avoids exposing credentials in YAML files
- allows reuse across multiple Pods

---

## Important Note (Security)

- Secrets are stored in base64 encoded format
- Base64 is NOT encryption → only obfuscation

This means:
- Anyone with access can decode it easily

For real security:
- Enable encryption at rest (e.g., KMS)
- Use RBAC to restrict access

---

## Secret Types

Common built-in types:
- Opaque → generic key-value Secret (default)
- kubernetes.io/basic-auth → username/password
- kubernetes.io/dockerconfigjson → image pull credentials
- kubernetes.io/tls → TLS cert + private key

---

## Creating Secrets

### From Command Line

kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

### From YAML

apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=
  password: c2VjcmV0MTIz

Note:
- Values must be base64 encoded

### Encoding Example

echo -n "admin" | base64

---

## Using Secrets in Pods

### As Environment Variables

env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: password

### Using envFrom

envFrom:
  - secretRef:
      name: my-secret

Imports all keys as environment variables

### As Files (Volume Mount)

volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
containers:
  - name: app
    image: my-app
    volumeMounts:
      - name: secret-volume
        mountPath: "/etc/secret"
        readOnly: true

Result:
- Each key becomes a file
- Example:
  - /etc/secret/password

---

## Image Pull Secrets

Used for private container registries:

kubectl create secret docker-registry my-registry-secret \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=email@example.com

Attach to Pod:

imagePullSecrets:
  - name: my-registry-secret

---

## How Updates Work

Environment Variables:
- Do NOT update automatically
- Require Pod restart

Volume Mounts:
- Updated automatically (eventual consistency)

---

## Security Best Practices

- Never commit Secrets to version control
- Use RBAC to limit access
- Enable encryption at rest (KMS)
- Use external secret managers if possible:
  - HashiCorp Vault
  - AWS Secrets Manager
- Prefer volume mounts over env vars (less exposure)

---

## Common Mistakes

- Assuming base64 = encryption
- Hardcoding secrets in YAML
- Printing secrets in logs
- Giving broad RBAC access

---

## Troubleshooting

Secret not found
- kubectl get secrets

Wrong value
- kubectl get secret my-secret -o yaml

Decode:
- echo "YWRtaW4=" | base64 --decode

Pod cannot access Secret
Check:
- correct name
- correct key
- same namespace

---

## Key Takeaways

- Secrets store sensitive data
- Base64 encoding ≠ encryption
- Can be used as:
  - environment variables
  - files (volumes)
- Env vars are static, volumes can update
- Must be protected with RBAC and encryption

---

## Useful Commands

Create Secrets:
- kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
- kubectl apply -f my-secret.yaml

List Secrets:
- kubectl get secrets

Describe Secret:
- kubectl describe secret my-secret

View YAML:
- kubectl get secret my-secret -o yaml

Delete:
- kubectl delete secret my-secret
