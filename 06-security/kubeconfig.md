# Kubeconfig

## Overview

The **kubeconfig** is a YAML configuration file used by **kubectl** and other Kubernetes clients to authenticate and communicate with the Kubernetes API server.

Without kubeconfig, you would need to manually specify for every command:

* API server address (`--server`)
* CA certificate (`--certificate-authority`)
* client certificate and key (`--client-certificate`, `--client-key`)

The kubeconfig file automates and centralizes this configuration.

---

## Kubeconfig Structure

A kubeconfig file consists of four main sections:

* clusters
* users
* contexts
* current-context

It also includes:

apiVersion: v1
kind: Config

---

## 1. clusters

Defines Kubernetes clusters (API server endpoints).

clusters:
- name: production
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority: /path/to/ca.crt

Alternative (embedded):

```
certificate-authority-data: <base64-encoded-ca.crt>
```

---

## 2. users

Defines authentication credentials.

users:
- name: prod-user
  user:
    client-certificate: /path/to/admin.crt
    client-key: /path/to/admin.key

Alternative (embedded):

```
client-certificate-data: <base64-encoded-cert>
client-key-data: <base64-encoded-key>
```

Other authentication methods also supported:

* token:
  token: <bearer-token>

* exec (OIDC / external auth plugins)

---

## 3. contexts

A context links:

* a cluster
* a user
* optionally a namespace

contexts:
- name: prod-user@production
  context:
    cluster: production
    user: prod-user
    namespace: default

---

## 4. current-context

Defines the default context used by kubectl.

current-context: prod-user@production

---

## Full Example

apiVersion: v1
kind: Config
clusters:
- name: production
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority: /path/to/ca.crt
users:
- name: prod-user
  user:
    client-certificate: /path/to/admin.crt
    client-key: /path/to/admin.key
contexts:
- name: prod-user@production
  context:
    cluster: production
    user: prod-user
    namespace: default
current-context: prod-user@production

---

## Where kubeconfig Is Located

Default location:

- $HOME/.kube/config

Custom file:

- kubectl get pods --kubeconfig=myconfig.yaml

Multiple files (merged):

- export KUBECONFIG=file1.yaml:file2.yaml

---

## How kubeconfig Is Used

### kubectl → API Server

1. kubectl reads kubeconfig
2. selects current-context
3. extracts:
   * cluster (server + CA)
   * user (credentials)
4. establishes TLS connection
5. authenticates request

---

## Important Notes

* kubeconfig can contain **multiple clusters and users**
* contexts allow easy switching between environments
* supports multiple authentication methods (cert, token, exec)

---

## Security Best Practices

* Never commit kubeconfig to Git if it contains:
  * private keys
  * certificates
  * tokens
* Prefer:
  * short-lived tokens (OIDC)
  * external authentication providers
* Restrict file permissions:
  * chmod 600 ~/.kube/config

---

## Best Practices

* Use separate contexts for environments (dev, staging, prod)
* Use meaningful naming (user@cluster)
* Avoid embedding sensitive data unless necessary
* Use KUBECONFIG for multi-cluster workflows

---

## Common Pitfalls

* Wrong current-context
* Expired certificates
* Missing CA configuration
* Mixing multiple kubeconfig files incorrectly
* Hardcoding sensitive data in version control

---

## Key Takeaways

* kubeconfig = central configuration for Kubernetes access
* abstracts connection + authentication details
* supports multiple clusters and users
* context determines active environment
* critical for kubectl and API communication

---

## Useful kubectl config Commands

View config:
- kubectl config view

Switch context:
- kubectl config use-context NAME

List contexts:
- kubectl config get-contexts

Create or modify context:
- kubectl config set-context NAME

Delete context:
- kubectl config delete-context NAME

Rename context:
- kubectl config rename-context OLD NEW

Set cluster or user:
- kubectl config set-cluster
- kubectl config set-credentials
