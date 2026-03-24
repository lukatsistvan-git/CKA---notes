# Gateway API

## Overview

The **Gateway API** is the next-generation Kubernetes networking API for managing **north-south (ingress) traffic**.

It is designed to address limitations of the traditional **Ingress** resource by providing:
* Better **extensibility**
* Clear separation of concerns
* Support for more protocols
* Improved multi-team and multi-tenant use cases

> Gateway API is developed by the Kubernetes SIG Network and is evolving as a standard replacement for many Ingress use cases.

---

## Why Gateway API?

### Limitations of Ingress

* Limited to **HTTP/HTTPS**
* Heavy reliance on **annotations** (implementation-specific)
* Poor separation of concerns
* Difficult to delegate configuration across teams
* Not easily extensible

---

## Gateway API Design Principles

Gateway API introduces a **role-oriented model**:
* **Infrastructure providers** → manage Gateways
* **Application developers** → define Routes
* **Platform teams** → define policies and classes

---

## Core Resources

### GatewayClass

Defines **which controller** manages Gateways.

apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: nginx.org/gateway-controller

#### Key Field

* `controllerName`
  * Identifies the controller implementation

> Multiple GatewayClasses can exist in a cluster.

---

### Gateway

Represents the **entry point** into the cluster.

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All

#### Key Fields

* `gatewayClassName`
  * Links to a GatewayClass

* `listeners`
  * Defines:
    * Protocol (HTTP, HTTPS, TCP, etc.)
    * Port
    * Hostnames (optional)

* `allowedRoutes`
  * Controls which Routes can attach:
    * `Same` → same namespace
    * `All` → all namespaces
    * `Selector` → label-based

---

### HTTPRoute

Defines **routing rules** (similar to Ingress rules).

apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
  namespace: default
spec:
  parentRefs:
    - name: my-gateway
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /app
      backendRefs:
        - name: my-app
          port: 80

#### Key Fields

* `parentRefs`
  * Connects the Route to a Gateway

* `matches`
  * Conditions:
    * Path
    * Headers
    * Query params
    * HTTP method

* `backendRefs`
  * Target Services

---

## Advanced Features

### Filters (Replacing Annotations)

Gateway API replaces annotations with **structured filters**.

#### URL Rewrite Example

rules:
  - matches:
      - path:
          type: PathPrefix
          value: /old
    filters:
      - type: URLRewrite
        urlRewrite:
          path:
            replacePrefixMatch: /new

Effect:
/old/* → /new/*

---

## TLS Support

Gateway API provides **native TLS configuration**.

### TLS Termination

listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
        - kind: Secret
          name: tls-secret

### Behavior

* Gateway receives HTTPS traffic
* Decrypts TLS at the edge
* Forwards plain HTTP to backend Services

---

### Benefits

* Standardized TLS configuration (no annotations)
* Centralized security point
* Supports multiple certificates and domains
* Integrates with tools like **cert-manager**

---

## Supported Protocols

Unlike Ingress, Gateway API supports:
* HTTP
* HTTPS
* TCP
* UDP (via other Route types like TCPRoute, UDPRoute)

---

## Additional Route Types

* **HTTPRoute** → HTTP/HTTPS traffic
* **TCPRoute** → TCP traffic
* **UDPRoute** → UDP traffic
* **TLSRoute** → TLS passthrough

---

## Traffic Flow

Client → Gateway → Route → Service → Pod

---

## Important Notes

* Gateway API is **not a drop-in replacement** for Ingress
* Requires a **Gateway Controller** implementation
* Still evolving (check version compatibility)

---

## Common Pitfalls

* Assuming Gateway API works without a controller
* Misconfiguring `allowedRoutes`
* Forgetting to bind Routes to Gateways
* Mixing Ingress and Gateway concepts incorrectly

---

## Best Practices

* Use Gateway API for new deployments
* Separate responsibilities:
  * Platform → Gateway
  * App teams → Routes
* Use structured filters instead of custom logic
* Centralize TLS at the Gateway layer

---

## Key Takeaways

* Gateway API is the **next evolution of Kubernetes ingress**
* Provides better structure and flexibility than Ingress
* Separates infrastructure and application concerns
* Supports multiple protocols and advanced routing
* Uses standardized, extensible configuration
