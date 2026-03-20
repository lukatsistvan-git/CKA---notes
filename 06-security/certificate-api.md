# Certificate API

## Overview

The Kubernetes **Certificate API** allows cluster entities (users, nodes, applications) to request TLS certificates signed by a **Certificate Authority (CA)**.

This enables:
* secure communication (TLS)
* identity verification
* integration with Kubernetes authentication

The main resource is the **CertificateSigningRequest (CSR)**.

---

## CertificateSigningRequest (CSR)

A CSR is a Kubernetes object used to request a certificate.

It contains:
* the certificate request (base64 encoded)
* intended usage (e.g. client auth)
* signer type
* status (Pending, Approved, Denied)

---

## CSR Workflow (Step-by-Step)

### 1. Generate Private Key

openssl genrsa -out jane.key 2048

* This is the user's private key
* Must remain secret

---

### 2. Generate Certificate Signing Request

openssl req -new 
-key jane.key 
-subj "/CN=jane" 
-out jane.csr

* Creates a CSR file
* CN (Common Name) identifies the user

---

### 3. Encode CSR to Base64

cat jane.csr | base64 | tr -d '\n'

---

### 4. Create CSR YAML

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

---

### 5. Create CSR in Cluster

- kubectl create -f jane-csr.yaml

---

## Managing CSR

### List Requests

- kubectl get csr

---

### Approve CSR

- kubectl certificate approve jane

---

### Deny CSR

- kubectl certificate deny jane

---

## Retrieving the Signed Certificate

Check CSR:
- kubectl get csr jane -o yaml

Look for:
- status:
- certificate: <base64-encoded-cert>

Decode:
- kubectl get csr jane -o jsonpath='{.status.certificate}' | base64 -d > jane.crt

Now you have:
- jane.key → private key
- jane.crt → signed certificate

---

## How It Works Internally

The **kube-controller-manager** handles certificate signing.

It:
* watches CSR objects
* signs approved requests
* uses the cluster CA

Required flags:
--cluster-signing-cert-file=/etc/kubernetes/ca.crt
--cluster-signing-key-file=/etc/kubernetes/ca.key

If these are missing:
- CSR will not be signed

---

## signerName (Important)

Defines which CA signs the certificate.

Common values:
* kubernetes.io/kube-apiserver-client
* kubernetes.io/kube-apiserver-client-kubelet
* kubernetes.io/kubelet-serving

Incorrect signerName:
* CSR will not be processed

---

## usages (Important)

Defines how the certificate can be used.

Common values:
* client auth
* server auth
* digital signature
* key encipherment

Must match intended purpose.

---

## CSR Status Flow

1. **Pending**
2. **Approved / Denied**
3. **Issued (certificate appears in status)**

---

## Using the Certificate

You can use the generated cert for:
* kubectl access (via kubeconfig)
* API authentication
* TLS client connections

Example kubeconfig fields:
* client-certificate: jane.crt
* client-key: jane.key

---

## Security Considerations

* Private keys must never be shared
* CSR approval should be controlled (RBAC!)
* Avoid auto-approving all CSRs
* Validate requester identity before approval

---

## Best Practices

* Use CSR API instead of manual signing
* Restrict who can approve CSRs
* Monitor CSR resources
* Use correct signerName and usages
* Rotate certificates regularly

---

## Common Pitfalls

* Forgetting base64 encoding
* Wrong signerName
* Missing usages
* CSR approved but not signed (controller misconfig)
* Missing CA flags in controller-manager

---

## Key Takeaways

* CSR = certificate request inside Kubernetes
* Approval is required before signing
* kube-controller-manager signs certificates
* signerName and usages are critical
* Used for secure authentication inside cluster

---

## Useful Commands

- kubectl get csr
- kubectl describe csr jane
- kubectl certificate approve jane
- kubectl certificate deny jane
- kubectl get csr jane -o yaml
