# Explore Environment

## Overview

Understanding **Linux networking fundamentals** is essential for Kubernetes, as cluster networking builds directly on these concepts.

This section covers:
* Basic networking (switches, routers, interfaces)
* Routing and packet forwarding
* DNS fundamentals
* Network namespaces and virtual networking
* Container networking basics (Docker)
* Foundations for Kubernetes networking

---

## Linux Networking Basics

### Switch vs Router

* **Switch (Layer 2)**

  * Connects devices within the same network
  * Uses **MAC addresses** to forward traffic

* **Router (Layer 3)**

  * Connects different networks
  * Uses **IP addresses** for routing

> Many home routers include an internal switch (LAN ports).

---

### Network Interfaces

* Represent network connection points (e.g., `eth0`, `eth1`, `wlan0`)
* A machine can have multiple interfaces:
  * Internal network (e.g., `eth0`)
  * External network (e.g., `eth1`)

Useful commands:

* Show interfaces:
  ip link

* Show IP addresses:
  ip addr

---

### Routing

Routing determines where packets are sent.

* View routing table:
  ip route

* Add route:
  ip route add 192.168.2.0/24 via 192.168.1.1

* Default route:
  ip route add default via 192.168.1.1
  
---

### IP Forwarding

Controls whether a machine can act as a router.

* Enable:
  echo 1 > /proc/sys/net/ipv4/ip_forward
  
* Disable:
  echo 0 > /proc/sys/net/ipv4/ip_forward
  
---

## DNS (Domain Name System)

### Purpose

DNS translates human-readable names into IP addresses.

Example:
* `google.com` → `142.250.x.x`

---

### Domain Structure

Example: `www.mail.example.com`
* `.` → root
* `com` → TLD
* `example` → domain
* `mail`, `www` → subdomains

---

### Common DNS Record Types

* **A** → IPv4 address
* **AAAA** → IPv6 address
* **CNAME** → alias
* **MX** → mail server
* **NS** → name server

---

### Name Resolution in Linux

Resolution order is defined in:

#### `/etc/nsswitch.conf`

hosts: files dns

Order:
1. `/etc/hosts`
2. DNS server

---

#### `/etc/hosts`

Static mappings:
* 127.0.0.1 localhost
* 192.168.1.100 nas.local

---

#### `/etc/resolv.conf`

Defines DNS servers:
* nameserver 8.8.8.8
* nameserver 1.1.1.1
* search example.com local

---

## CoreDNS

### Overview

* Default DNS server in Kubernetes
* Provides **service discovery**
* Plugin-based architecture
* Configured via **Corefile**

---

### Example Corefile

. {
    forward . 8.8.8.8
    log
    errors
}

---

### Common Plugins

* `kubernetes` → resolves Services and Pods
* `forward` → forwards queries
* `hosts` → static entries
* `log` → logs queries
* `errors` → logs errors

---

### Kubernetes DNS Behavior

* Pods use DNS for service discovery

* Example:
  ping my-service

* Fully qualified name:
  <service>.<namespace>.svc.cluster.local

---

## Network Namespaces

### Overview

A **network namespace** provides isolated networking:
* Interfaces
* IP addresses
* Routing tables
* iptables rules

Used by containers and Kubernetes Pods.

---

### Create Namespaces

* ip netns add red
* ip netns add blue

---

### Connect Namespaces (veth pair)

* ip link add veth-red type veth peer name veth-blue
* ip link set veth-red netns red
* ip link set veth-blue netns blue

Assign IPs:
* ip -n red addr add 192.168.15.1/24 dev veth-red
* ip -n blue addr add 192.168.15.2/24 dev veth-blue

Enable interfaces:
* ip -n red link set veth-red up
* ip -n blue link set veth-blue up
* ip -n red link set lo up
* ip -n blue link set lo up

---

## Linux Bridge (Virtual Switch)

Acts like a virtual switch connecting namespaces.

### Create Bridge
* ip link add v-net-0 type bridge
* ip link set v-net-0 up

Connect veth to bridge:
* ip link set veth-red-br master v-net-0

---

## Internet Access via Host (NAT)

### Default Gateway in Namespace

* ip netns exec red ip route add default via 192.168.15.254

---

### Enable IP Forwarding

* echo 1 > /proc/sys/net/ipv4/ip_forward

---

### Configure NAT (MASQUERADE)

* iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -o eth0 -j MASQUERADE

---

### Port Forwarding (DNAT)

* iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.15.1:80

---

## Docker Networking

Docker uses Linux networking primitives under the hood.

---

### Network Modes

#### 1. none

* No networking
* No interfaces

---

#### 2. host

* Shares host network
* No isolation
* No NAT

---

#### 3. bridge (default)

* Creates `docker0` bridge
* Each container gets:
  * Network namespace
  * veth pair
  * IP (e.g., `172.17.0.x`)

---

### Outbound Traffic

* Uses **NAT (MASQUERADE)**
* Traffic appears from host IP

---

### Inbound Traffic (Port Mapping)

docker run -p 8080:80 nginx

* Maps host port → container port
* Uses **DNAT via iptables**

---

## Relevance to Kubernetes

Kubernetes networking builds on these primitives:
* Pods use **network namespaces**
* CNI plugins configure networking
* Each Pod gets:
  * Unique IP
  * Flat network (no NAT between Pods)

Key Kubernetes requirements:
* Pod-to-Pod communication without NAT
* Node-to-Pod communication
* DNS-based service discovery (CoreDNS)

---

## Common Pitfalls

* Misconfigured routing tables
* IP forwarding disabled
* Missing NAT rules
* Incorrect DNS configuration
* Overlapping IP ranges

---

## Key Takeaways

* Linux networking fundamentals are critical for Kubernetes
* Namespaces provide isolation
* Bridges and veth pairs connect workloads
* NAT enables external communication
* CoreDNS enables service discovery
* Kubernetes networking is built on these concepts

---

## Useful Commands

ip link
ip addr
ip route
ip netns
iptables -t nat -L
cat /etc/resolv.conf
