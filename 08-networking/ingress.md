# Ingress

## Overview

An **Ingress** is a Kubernetes resource that manages **external HTTP/HTTPS access** to Services داخل a cluster.

It provides:
* Routing based on **hostnames and paths**
* TLS (HTTPS) termination
* Centralized entry point for multiple Services

> Ingress itself does **not handle traffic** — it only defines rules.

---

## Ingress vs Ingress Controller

### Ingress

* Kubernetes API object (`kind: Ingress`)
* Defines routing rules:
  * Hostnames
  * Paths
  * Backend Services

---

### Ingress Controller

An **Ingress Controller** is a component that:
* Watches Ingress resources
* Interprets rules
* Configures a proxy/load balancer
* Routes external traffic to Services

Common implementations:
* NGINX Ingress Controller
* Traefik
* HAProxy
* Cloud-specific controllers (AWS ALB, GCE)

---

## Why Use Ingress

Ingress allows you to:
* Expose multiple Services via a **single IP or domain**
* Route traffic based on:
  * URL path (`/app1`, `/api`)
  * Host (`app.example.com`)
* Enable **TLS (HTTPS)**
* Perform **path rewrites**
* Centralize traffic management

---

## How It Works

Typical flow:
1. Application runs in Pods
2. A **Service** exposes the Pods
3. An **Ingress Controller** is deployed
4. An **Ingress resource** is created with rules
5. External traffic flows:

Client → Ingress Controller → Service → Pod

---

## Ingress Controller Deployment

* Usually deployed as a **Deployment or DaemonSet**
* Exposed via:
  * **LoadBalancer Service** (cloud environments)
  * **NodePort Service** (bare metal)

---

## Example Ingress Resource

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80

---

## Explanation

* `host: myapp.example.com`
  * Matches incoming requests by hostname

* `path: /app1`
  * Matches URL path

* `pathType: Prefix`
  * Matches all paths starting with `/app1`

* `backend`
  * Defines target Service

---

## Path Rewrite

Annotation:

nginx.ingress.kubernetes.io/rewrite-target: /

Effect:

* Request:
  /app1 → /

Useful when backend apps expect root (`/`) paths.

---

## TLS Configuration

Ingress supports HTTPS via TLS:

spec:
  tls:
    - hosts:
        - myapp.example.com
      secretName: my-tls-secret

* TLS certificate stored in a **Secret**
* Ingress Controller handles termination

---

## Important Notes

* Ingress works only for:
  * **HTTP**
  * **HTTPS**
* Not suitable for:
  * TCP/UDP (use Service type LoadBalancer or other solutions)

---

## Default Backend

* Handles unmatched requests
* Returns:
  * 404 (usually)

---

## Ingress Class

Used to specify which controller should handle the Ingress:

spec:
  ingressClassName: nginx

* Required in clusters with multiple controllers

---

## Annotations

Used to customize controller behavior.

Examples (NGINX-specific):
* Rewrite rules
* SSL redirect
* Rate limiting
* Authentication

> Annotations are **controller-specific**, not part of core Kubernetes API.

---

## Common Pitfalls

* Forgetting to install an Ingress Controller
* Misconfigured annotations
* Missing DNS records for hostnames
* Using Ingress for non-HTTP traffic
* Not specifying `ingressClassName` in multi-controller setups

---

## Best Practices

* Always deploy a production-ready Ingress Controller
* Use TLS for all external endpoints
* Use host-based routing for clarity
* Keep Ingress resources simple and modular
* Avoid overusing annotations (prefer clear configs)

---

## Key Takeaways

* Ingress defines **routing rules**, not traffic handling
* Ingress Controller executes those rules
* Enables HTTP/HTTPS exposure with a single entry point
* Supports host/path-based routing and TLS
* Central component for external access in Kubernetes
