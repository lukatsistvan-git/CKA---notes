# Certificates

## Overview

Kubernetes uses **TLS certificates** to secure communication between:
- users and the API server
- cluster components
- nodes

This provides:
- **encryption (confidentiality)**
- **authentication (identity verification)**
- **integrity (data is not modified)**

At the core of this system is a **PKI (Public Key Infrastructure)** based on a **Certificate Authority (CA)**.

---

## Kubernetes Security Model (High-Level)

Kubernetes security is built on multiple layers:
- **Authentication** → Who are you?
- **Authorization** → What can you do?
- **TLS Communication** → Is communication secure?

The **kube-apiserver** is the central control point:
- validates identity
- enforces access control
- serves as the secure entry point

---

## Authentication Types (Context)

Kubernetes supports multiple authentication mechanisms:
- **X.509 certificates** (primary for components)
- **Bearer tokens** (ServiceAccounts, OIDC)
- **OpenID Connect (OIDC)**
- **Webhook authentication**

Deprecated:
- `--basic-auth-file`
- `--token-auth-file`

---

## TLS Basics (Simplified Flow)

### TLS Handshake

1. Client connects to server (HTTPS)
2. Server sends its **certificate**
3. Client verifies:
   * trusted CA
   * valid signature
   * correct hostname
   * expiration
4. Client generates a **session key**
5. Key is encrypted using server’s public key
6. Server decrypts using private key
7. Communication continues using **symmetric encryption**

---

## Mutual TLS (mTLS) in Kubernetes

Kubernetes uses **mutual TLS**:
- **Client verifies server**
- **Server verifies client**

Used in:
- kubelet ↔ API server
- controller ↔ API server
- scheduler ↔ API server

---

## Certificate Types in Kubernetes

### 1. Certificate Authority (CA)

The CA signs all certificates.

Files:
- `ca.crt`
- `ca.key`

Optional separate CAs:
- `etcd-ca.crt`
- `front-proxy-ca.crt`

---

### 2. Server Certificates

Used by components that **receive requests**:
- kube-apiserver
- etcd
- kubelet

Examples:
- `kube-apiserver.crt / key`
- `etcd-server.crt / key`
- `kubelet.crt / key`

---

### 3. Client Certificates

Used by components that **connect to others**:
- admin user
- scheduler
- controller-manager
- kube-proxy

Examples:
- `admin.crt / key`
- `scheduler.crt / key`
- `controller-manager.crt / key`
- `kube-proxy.crt / key`

Special cases:
- `apiserver-kubelet-client.crt`
- `apiserver-etcd-client.crt`

---

## Where Certificates Are Stored

### kubeadm Clusters

- Certificates:
  `/etc/kubernetes/pki/`

- kubeconfig (admin access):
  `/etc/kubernetes/admin.conf`

- kubelet certs:
  `/var/lib/kubelet/pki/`

---

## How Certificates Are Used (Flow Example)

### kubectl → API Server

1. kubectl uses **kubeconfig**
2. kubeconfig contains:
   - client certificate
   - CA certificate
   - API server endpoint
3. API server:
   - verifies client cert
   - extracts identity (CN)
   - passes to authorization (RBAC)

---

## Generating Certificates (Manual - OpenSSL)

### Step 1: Create CA

Private key:
- openssl genrsa -out ca.key 4096

Certificate:
- openssl req -x509 -new -nodes -key ca.key -subj "/CN=kubernetes-ca" -days 1000 -out ca.crt

---

### Step 2: Create Component Certificate

Private key:
- openssl genrsa -out apiserver.key 2048

CSR:
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr

---

### Step 3: Sign Certificate

openssl x509 -req 
-in apiserver.csr 
-CA ca.crt 
-CAkey ca.key 
-CAcreateserial 
-out apiserver.crt 
-days 365 
-extensions v3_req 
-extfile apiserver-ext.cnf

---

## Subject Alternative Names (SAN)

Required for components accessed via multiple endpoints.

Example config:
[ v3_req ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 127.0.0.1

Without SAN:
- certificate validation will fail

---

## kubeadm and Certificates

kubeadm automatically:

- generates CA
- creates all required certificates
- configures components

Useful commands:
- kubeadm certs check-expiration
- kubeadm certs renew all

---

## Important Concepts

### CN (Common Name)

- identifies the entity (user/component)
- used in authentication

---

### Private vs Public Key

- **Private key** → must never leave the node
- **Public cert** → shared

---

### Certificate Signing

- CA signs certificates
- ensures trust across components

---

## Best Practices

- Rotate certificates regularly
- Protect private keys
- Use separate CAs for etcd if needed
- Always include SAN
- Prefer kubeadm automation

---

## Common Pitfalls

- Missing SAN → TLS errors
- Expired certificates → cluster failure
- Wrong CN → authentication issues
- Mixing client/server cert roles
- Losing CA key → cannot issue new certs

---

## Key Takeaways

- Kubernetes relies heavily on **TLS + PKI**
- **mTLS is default** between components
- CA is the root of trust
- kubeadm simplifies certificate management
- Certificates directly impact authentication

---

## Useful Commands

- kubeadm certs check-expiration
- kubeadm certs renew all
- openssl x509 -in cert.crt -text -noout
- kubectl config view
