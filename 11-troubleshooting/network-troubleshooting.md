# Network Troubleshooting

## Overview

Networking in Kubernetes enables communication between:
* Pods ↔ Pods
* Pods ↔ Services
* External clients ↔ Cluster (NodePort / Ingress)

Typical flow:
* Client → Service → Pod → Container

Common symptoms of network issues:
* Connection timeouts
* DNS resolution failures
* Service unreachable

---

## Troubleshooting Strategy

### 1. Start from the Client Perspective

Test external access:
* curl http://<service-ip>:<port>

NodePort:
* curl http://<node-ip>:<node-port>

Ingress:
* curl http://<domain>

If it fails → move inward step by step

---

## Step-by-Step Debugging

---

### 2. Check Service

* kubectl get svc  
* kubectl describe svc <service-name>

#### What to Verify

* ClusterIP / NodePort assigned
* Correct port mapping (targetPort vs port)
* Selector matches Pods

---

### Endpoints Check

* kubectl get endpoints <service-name>

If `ENDPOINTS = <none>`:
* Service is NOT linked to Pods

---

### Label Matching

* kubectl get pods --show-labels

Ensure:
* Service selector == Pod labels

---

## 3. Check Pod Status

* kubectl get pods -o wide

Verify:
* Pod is Running
* Pod IP exists
* Correct node assignment

---

## 4. Test Pod-to-Pod Connectivity

* kubectl exec -it <pod> -- ping <pod-ip>  
* kubectl exec -it <pod> -- curl http://<pod-ip>:<port>

If fails:
* CNI issue
* NetworkPolicy blocking

---

## 5. Check DNS Resolution

* kubectl exec -it <pod> -- nslookup <service-name>  
* kubectl exec -it <pod> -- dig <service-name>.default.svc.cluster.local

If DNS fails:
* kubectl get pods -n kube-system

Look for CoreDNS pods:
* Not Running
* CrashLoopBackOff

---

## 6. Check CoreDNS Logs

* kubectl logs -n kube-system <coredns-pod>

Look for:
* upstream DNS failures
* config errors

---

## 7. Test Service Connectivity from Pod

* kubectl exec -it <pod> -- curl http://<service-name>:<port>

If fails:
* Service misconfiguration
* kube-proxy issue
* NetworkPolicy blocking

---

## 8. Check kube-proxy

kube-proxy manages Service networking (iptables/ipvs).

* kubectl get pods -n kube-system

Or on node:

* sudo systemctl status kube-proxy

If kube-proxy fails:
* Services will not route traffic

---

## 9. Check Network Policies

* kubectl get networkpolicy  
* kubectl describe networkpolicy <policy-name>

Look for:
* Deny rules blocking traffic
* Missing allow rules

Test by temporarily removing policy if needed

---

## 10. Check CNI Plugin

CNI provides Pod networking.

* kubectl get pods -n kube-system

Look for:
* calico / flannel / weave pods

If failing:
* Pod-to-Pod communication breaks

---

## 11. Node-Level Network Checks

SSH into node:

* ip a  
* ip route  
* ping <other-node-ip>

Check:
* Interface exists
* Routes are correct
* Nodes can reach each other

---

## 12. Check Ports and Firewall

Ensure required ports are open.

Test:
* ss -tulnp

Common issues:
* Firewall blocking traffic
* Closed NodePort range

---

## Common Failure Scenarios

### Service Has No Endpoints

* Cause: label mismatch
* Fix: align selectors and labels

---

### DNS Not Working

* Cause: CoreDNS failure
* Fix: restart or debug CoreDNS

---

### Pod-to-Pod Communication Fails

* Cause: CNI issue
* Fix: check CNI plugin

---

### Cannot Reach Service

* Cause:
  * kube-proxy issue
  * NetworkPolicy blocking
* Fix:
  * verify rules and logs

---

### Ingress Not Working

* Cause:
  * controller not running
  * wrong routing config
* Fix:
  * check ingress controller logs

---

## Debugging Workflow Summary

1. Client request (curl)
2. Service (selectors, endpoints)
3. Pod (status, IP)
4. Pod-to-Pod connectivity
5. DNS (CoreDNS)
6. Service access from Pod
7. kube-proxy
8. NetworkPolicy
9. CNI plugin
10. Node network

---

## Best Practices

* Use consistent labels
* Keep CNI plugin healthy
* Monitor CoreDNS
* Avoid overly restrictive NetworkPolicies
* Validate connectivity from inside Pods

---

## Key Takeaways

* Always debug from outside → inside
* Most network issues come from:
  * label mismatch
  * DNS failure
  * CNI problems
  * NetworkPolicies
* Use kubectl + in-pod testing for accurate results
