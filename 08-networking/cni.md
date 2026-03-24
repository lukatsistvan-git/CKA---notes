# CNI (Container Network Interface)

## Overview

The **Container Network Interface (CNI)** is a standardized interface used by container runtimes (such as Kubernetes via CRI, containerd, CRI-O) to configure networking for containers.

CNI provides a **modular and extensible framework** for connecting containers to networks.

> Kubernetes does **not implement networking itself** — it relies on CNI plugins.

---

## How CNI Works

When a Pod is created:

1. The container runtime creates the container
2. The runtime calls the **CNI plugin**
3. The plugin:
   * Assigns an IP address
   * Connects the Pod to the network
   * Configures routes and interfaces
4. The Pod becomes reachable in the cluster network

---

## CNI Plugins

CNI uses plugins to implement networking behavior.

Common plugins:

* **Flannel**

  * Simple overlay network
  * Easy to set up
  * Does **not support NetworkPolicy**

* **Calico**

  * Layer 3 networking
  * High performance and scalable
  * Supports **NetworkPolicy**

* **Cilium**

  * eBPF-based networking
  * Advanced observability and security
  * Supports **NetworkPolicy**

* **Weave Net**

* **Kube-Router**

---

## Network Policy Support

Not all CNI plugins support **NetworkPolicy**.

* Supported: Calico, Cilium
* Not supported: Flannel (by default)

> If the CNI plugin does not support NetworkPolicy, Kubernetes will **ignore those rules**.

---

## CNI vs CNM

* **CNI (Container Network Interface)**

  * Used by Kubernetes
  * Plugin-based
  * Industry standard

* **CNM (Container Network Model)**

  * Docker-specific model
  * Used by Docker networking drivers

Today, **CNI is the dominant standard** in Kubernetes environments.

---

## Cluster Networking

### Overview

In Kubernetes, all nodes (control plane and worker nodes) must be able to communicate reliably.

Networking is **flat and IP-based**.

---

## Core Networking Requirements (Kubernetes Model)

According to Kubernetes networking principles:
* Every **Pod gets its own IP address**
* Pods can communicate with **all other Pods without NAT**
* Nodes can communicate with all Pods
* Agents (like kubelet) can communicate with the control plane

---

## Node Requirements

* Each node must have:
  * At least one active network interface (e.g., `eth0`)
  * Unique IP address and hostname
* Nodes must reach each other **directly via IP (no NAT in between)**

---

## Pod Networking

* Each Pod:
  * Has its own **network namespace**
  * Gets a unique IP address
* Containers inside a Pod:
  * Share the same network namespace
  * Communicate via `localhost`

---

## Cluster Networking Responsibilities

Handled by the CNI plugin:
* IP address allocation for Pods
* Routing between Pods across nodes
* Network connectivity
* (Optional) NetworkPolicy enforcement

---

## Important Ports

Kubernetes control plane and node components use specific ports:
* **6443** → Kubernetes API Server
* **2379–2380** → etcd
* **10250** → kubelet
* **10257** → kube-controller-manager
* **10259** → kube-scheduler

---

### NodePort Range

* **30000–32767**

  * Default range for NodePort Services
  * Exposes services externally via node IP

---

## Common Pitfalls

* Using a CNI plugin without NetworkPolicy support
* Overlapping Pod CIDR ranges
* Missing routes between nodes
* Firewall blocking required ports
* NAT between nodes (breaks Kubernetes networking model)

---

## Best Practices

* Choose a CNI plugin based on requirements:
  * Simplicity → Flannel
  * Security → Calico / Cilium
* Ensure full **Pod-to-Pod connectivity without NAT**
* Validate required ports are open
* Use NetworkPolicies where supported

---

## Key Takeaways

* CNI is the **networking backbone of Kubernetes**
* Kubernetes delegates networking to CNI plugins
* Every Pod gets a unique IP in a flat network
* Not all CNI plugins support NetworkPolicy
* Proper networking setup is critical for cluster functionality

---

## Useful Linux Networking Commands

* List interfaces:
  ip link

* Show IP addresses:
  ip addr

* Add IP address:
  ip addr add 192.168.1.10/24 dev eth0

* Show routing table:
  ip route

* Add route:
  ip route add 192.168.1.0/24 via 192.168.2.1

* Enable IP forwarding:

  echo 1 > /proc/sys/net/ipv4/ip_forward

* Show ARP table:
  arp

* Show listening ports:
  netstat -plnt
