# Service Networking

## Overview

In Kubernetes, **Pods are ephemeral** — their IP addresses can change over time.
Because of this, directly communicating with Pod IPs is unreliable.

**Services** solve this problem by providing a **stable network abstraction**.

---

## Why Services Are Needed

Problems with Pods:

* Pod IPs are **dynamic**
* Pods can be:
  * Restarted
  * Rescheduled to another node
  * Scaled up/down

Solution:

* Use a **Service** instead of Pod IPs

A Service provides:

* A **stable IP address (ClusterIP)**
* A **stable DNS name**
* Load balancing across Pods

---

## What is a Service?

A **Service** is an abstraction that:

* Groups Pods using **labels/selectors**
* Exposes them via:
  * Stable IP
  * DNS name
* Distributes traffic to matching Pods

---

## Service Types

* **ClusterIP (default)**
  * Internal access only

* **NodePort**
  * Exposes Service on node ports

* **LoadBalancer**
  * Integrates with cloud providers

* **ExternalName**
  * Maps to external DNS (no ClusterIP)

> Only **ExternalName** does NOT get a ClusterIP.

---

## ClusterIP Range

Each Service gets an IP from a predefined range.

Configured in the API server:
--service-cluster-ip-range=10.96.0.0/12

This means:

* Service IP range:
  10.96.0.0 – 10.111.255.255

* Each Service gets **one unique IP**

* This range is **separate from Pod CIDR**

---

## How Service IP is Assigned

When a Service is created:
1. Kubernetes checks the configured IP range
2. Allocates the **next available IP**
3. Assigns it to the Service

* The IP is tied to the Service lifecycle
* When deleted → IP can be reused

---

### Manual ClusterIP Assignment

You can specify it manually:

spec:
  clusterIP: 10.96.0.10

* If the IP is already in use → Service creation fails

---

## DNS for Services

Each Service gets a DNS entry:

<service-name>.<namespace>.svc.cluster.local

Example:
* curl my-service.default.svc.cluster.local

DNS is provided by **CoreDNS**.

---

## kube-proxy

### Role

**kube-proxy** is responsible for:
* Implementing Service networking
* Routing traffic from Service IP → Pod IPs

It runs on every node (DaemonSet).

---

## kube-proxy Modes

kube-proxy supports multiple proxy modes:

---

### 1. iptables (default)

* Uses **Linux iptables**
* Implements NAT and forwarding rules

Characteristics:
* Kernel-level packet handling
* Widely supported
* Stable and reliable

Pros:
* Simple and well-tested

Cons:
* Large number of rules can be harder to manage

---

### 2. IPVS (recommended for large clusters)

* Uses **IP Virtual Server (IPVS)**

Characteristics:
* Better performance than iptables
* Supports advanced load balancing:
  * Round Robin (RR)
  * Least Connections (LC)
  * Weighted RR (WRR)

Pros:
* Scales better
* More efficient connection handling

Cons:
* Requires IPVS kernel modules

---

### 3. userspace (deprecated)

* kube-proxy handles traffic in userspace

Characteristics:
* Slow
* Not scalable

> Not recommended for production

---

## How kube-proxy Works

When a Service is created:
1. kube-proxy watches the API server
2. Creates rules (iptables/IPVS)
3. Traffic to Service IP is redirected to backend Pods

Example flow:
* Client → Service IP → kube-proxy → Pod IP

---

## Important Notes

* Service IPs are **virtual IPs (VIPs)**, not bound to a real interface
* kube-proxy handles traffic redirection
* Load balancing is done at **L4 (TCP/UDP)**

---

## Verifying kube-proxy Mode

Check configuration:
* kubectl get pod -n kube-system -l k8s-app=kube-proxy -o yaml | grep proxy-mode

Or logs:
* kubectl logs -n kube-system <kube-proxy-pod>

---

## Common Pitfalls

* Confusing Pod IP with Service IP
* Overlapping Service CIDR with Pod CIDR
* Manually assigning conflicting ClusterIP
* Not understanding kube-proxy behavior
* Assuming Service creates actual network interfaces

---

## Best Practices

* Use default ClusterIP for internal communication
* Avoid manual IP assignment unless necessary
* Use IPVS mode for large clusters
* Monitor kube-proxy performance
* Rely on DNS instead of hardcoding IPs

---

## Key Takeaways

* Services provide **stable networking for Pods**
* ClusterIP is the internal virtual IP
* kube-proxy implements traffic routing
* IPVS is preferred for large-scale environments
* DNS (CoreDNS) enables service discovery
