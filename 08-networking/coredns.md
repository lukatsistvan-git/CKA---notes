# CoreDNS

## Overview

In Kubernetes, **DNS provides service discovery** by translating logical names into IP addresses.

Instead of using Pod or Service IPs directly, workloads communicate using **DNS names**.

Kubernetes uses **CoreDNS** (default) to implement this functionality.

---

## DNS in Kubernetes

### Purpose

DNS enables:
* Pod-to-Service communication using names
* Decoupling from dynamic IP addresses
* Scalable service discovery

Example:
* my-service.default.svc.cluster.local → ClusterIP

---

## DNS Components

* **CoreDNS**
  * Default DNS server in Kubernetes

* **kube-dns Service**
  * Exposes CoreDNS داخل the cluster

* **Pod `/etc/resolv.conf`**
  * Configured to use cluster DNS

---

## DNS Resolution in Pods

Each Pod gets a DNS configuration like:

nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local

* `nameserver` → CoreDNS Service IP
* `search` → enables short names (e.g., `my-service`)

---

## Service DNS Names

### Fully Qualified Domain Name (FQDN)

<service>.<namespace>.svc.cluster.local

Example:
* curl my-service.default.svc.cluster.local

---

### Short Names

Within the same namespace:
* curl my-service

Across namespaces:
* curl my-service.other-namespace

---

## DNS Record Types in Kubernetes

### A Record

* Maps Service name → ClusterIP
* Most common

Example:
* my-service.default.svc.cluster.local → 10.96.0.X

---

### CNAME Record

* Alias to another domain
* Used by **ExternalName Services**

---

## Domain Name Hierarchy

Example:
* my-service.default.svc.cluster.local

Breakdown:
* `my-service` → Service name
* `default` → Namespace
* `svc` → Service category
* `cluster.local` → Cluster domain (default)

> The cluster domain can be customized (e.g., `--cluster-domain`).

---

## CoreDNS Architecture

### Corefile

CoreDNS is configured via the **Corefile**.

Location inside Pod:
/etc/coredns/Corefile

---

### Example Corefile

.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}

---

## CoreDNS Plugins

### kubernetes Plugin

* Integrates with Kubernetes API
* Resolves:
  * Services
  * Pods (optional)
  * Namespaces

Key options:

* `cluster.local` → cluster domain
* `in-addr.arpa`, `ip6.arpa` → reverse DNS

---

### pods insecure

* Allows resolving Pod IP-based DNS names
* Example:
  10-244-2-5.namespace.pod.cluster.local
  
Not recommended for production:
* Skips security validation
* Can bypass DNS-based isolation

---

### Other Common Plugins

* `forward`
  * Forwards external queries (e.g., to `/etc/resolv.conf`)

* `cache`
  * Caches DNS responses

* `log`, `errors`
  * Observability

* `prometheus`
  * Metrics endpoint

* `reload`
  * Auto-reload config

* `loadbalance`
  * Randomizes response order

---

## CoreDNS Configuration (ConfigMap)

* Stored as a **ConfigMap**:
kubectl -n kube-system get configmap coredns -o yaml

> CoreDNS Pods mount this ConfigMap

---

### Applying Changes

After modifying CoreDNS config:
kubectl rollout restart deployment coredns -n kube-system

> Required to reload configuration

---

## kube-dns Service

CoreDNS runs behind a Service:

NAME       TYPE        CLUSTER-IP
kube-dns   ClusterIP   10.96.0.10

* Selector:
  k8s-app=kube-dns

* This IP is injected into all Pods as the DNS server

---

## DNS Query Tools

You can test DNS inside a Pod:
* `nslookup`
* `dig`
* `host`

Example:
* dig my-service.default.svc.cluster.local

---

## Important Notes (Kubernetes.io Alignment)

* DNS is **cluster-scoped service discovery**
* CoreDNS replaces legacy **kube-dns**
* DNS records are dynamically updated based on cluster state
* Services always get DNS entries
* Pods only get DNS entries if configured (plugin-dependent)

---

## Common Pitfalls

* DNS not working due to CoreDNS failure
* Misconfigured Corefile
* Missing kube-dns Service
* Using FQDN unnecessarily (when short name works)
* Relying on Pod DNS without proper plugin support

---

## Best Practices

* Use Service DNS names instead of IPs
* Avoid enabling `pods insecure` in production
* Monitor CoreDNS performance
* Use short names within the same namespace
* Validate DNS with `dig` or `nslookup`

---

## Key Takeaways

* CoreDNS provides DNS-based service discovery
* Services are reachable via stable DNS names
* DNS configuration is injected into every Pod
* CoreDNS is highly extensible via plugins
* DNS is critical for Kubernetes communication
