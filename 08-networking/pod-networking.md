# Pod Networking

## Overview

Kubernetes **does not implement Pod networking directly**, but defines a clear networking model that must be satisfied by the cluster.

This model is implemented by a **CNI plugin** (e.g., Calico, Flannel, Cilium, Weave).

---

## Kubernetes Networking Model

Kubernetes requires the following:

* Every **Pod has a unique IP address**
* Pods can communicate with each other:
  * On the **same node**
  * Across **different nodes**
* Communication happens **without NAT**
* The network is **flat and IP-based**
* No need for port mapping between Pods

---

## Key Principles

* Pod IPs are routable within the cluster
* Containers in the same Pod:

  * Share the same network namespace
  * Communicate via `localhost`
* Pod-to-Pod communication must be **direct**

---

## Example Cluster Networking

Assume a 3-node cluster:

* `node1` → `192.168.1.11`
* `node2` → `192.168.1.12`
* `node3` → `192.168.1.13`

Each node:

* Has a **Pod CIDR range**, for example:

  * `node1` → `10.244.1.0/24`
  * `node2` → `10.244.2.0/24`
  * `node3` → `10.244.3.0/24`

* Uses a **bridge interface** (implementation-specific)

  * Example: `cni0` (commonly used instead of custom names like `v-net-0`)

* Bridge IP acts as **default gateway**

  * Example: `10.244.1.1`

> The exact implementation depends on the CNI plugin.

---

## How Pod Networking Works (Under the Hood)

When a Pod is created:
1. The container runtime creates a **network namespace**
2. A **veth pair** is created:
   * One end inside the Pod
   * One end on the host
3. The host-side interface is attached to a **bridge or virtual network**
4. The Pod interface:
   * Gets an IP address (e.g., `10.244.1.12`)
   * Gets a default route via the bridge
5. Interfaces are brought **UP**

---

### Manual Equivalent (Simplified)

# Create veth pair
ip link add veth0 type veth peer name veth1

# Attach to bridge
ip link set veth0 master cni0
ip link set veth0 up

# Move to namespace
ip link set veth1 netns <namespace>
ip -n <namespace> addr add 10.244.1.X/24 dev veth1
ip -n <namespace> link set veth1 up
ip -n <namespace> route add default via 10.244.1.1

> In real clusters, this is handled automatically by the CNI plugin.

---

## Role of CNI

CNI is responsible for:
* Creating network interfaces
* Assigning IP addresses
* Connecting Pods to the network
* Setting up routing

---

### CNI Execution Model

* Plugins are located in:
  /opt/cni/bin

* Configurations are stored in:
  /etc/cni/net.d/

* When a Pod is created:
  * The container runtime calls the CNI plugin
  * The plugin receives configuration via **JSON (stdin)**
  * Executes actions:
    * `ADD` → create networking
    * `DEL` → clean up

---

## Inter-Node Communication

Pods on different nodes communicate via:
* **Routing** (Calico-style, L3)
* **Overlay networks** (Flannel, Weave)

---

### Overlay Networking

* Encapsulates traffic (e.g., VXLAN)
* Allows Pods to communicate across nodes
* Creates a **virtual flat network**

---

## Example: Weave CNI

### Overview

* Provides an **overlay network**
* Enables Pod communication across nodes
* Uses **VXLAN or similar tunneling**

---

### Deployment

* Installed as a **DaemonSet**
* Runs one Pod per node

Each Weave Pod:
* Acts as a network agent
* Connects to other nodes
* Configures networking locally

---

### Behavior

* Forms a **peer-to-peer network**
* Uses **automatic peer discovery**
* Can encrypt traffic
* Provides a unified Pod network

---

## IP Address Management (IPAM)

### Overview

IPAM is responsible for:
* Assigning IP addresses to Pods
* Avoiding IP conflicts
* Tracking allocated vs available IPs

---

### Kubernetes Perspective

* IPAM is handled by the **CNI plugin**
* Kubernetes itself does not manage Pod IP allocation

---

### Example: Weave IPAM

* Uses a **decentralized model**
* Each node:
  * Allocates IPs locally
  * Synchronizes with peers
* No central controller required

---

## Important Notes (Kubernetes.io Alignment)

* Kubernetes does **not mandate implementation details**
* Only defines the **networking requirements**
* CNI plugins decide:
  * How routing works
  * Whether overlay or native routing is used
  * How IPAM is implemented

---

## Common Pitfalls

* Assuming Kubernetes manages networking internally
* Misconfigured Pod CIDR ranges
* Overlapping IP ranges across nodes
* Missing routes between nodes
* Using CNI without proper IPAM configuration

---

## Best Practices

* Use a production-grade CNI plugin (Calico, Cilium)
* Ensure Pod CIDRs do not overlap
* Validate Pod-to-Pod connectivity across nodes
* Understand your CNI’s networking model (overlay vs routing)
* Monitor IP allocation (IPAM)

---

## Key Takeaways

* Pod networking is implemented by CNI, not Kubernetes
* Each Pod gets a unique IP and full network access
* Networking is flat and NAT-free between Pods
* veth pairs and namespaces form the foundation
* CNI plugins handle routing, IPAM, and connectivity
