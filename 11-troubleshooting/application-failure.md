# Application Failure Troubleshooting

## Overview

Troubleshooting application failures in Kubernetes requires a **systematic, layered approach**.

A typical application stack:
* User → Service → Pod → Container → Dependency (e.g., DB)

When an issue occurs, always **start from the outside (user perspective)** and move inward.

---

## Troubleshooting Strategy

### 1. Start from the User Perspective

Test external accessibility:
* curl http://<node-ip>:<node-port>

Or if using Ingress:
* curl http://<domain>

If this fails → move step by step inward.

---

## Step-by-Step Debugging

---

### 2. Check Service

* kubectl get svc
* kubectl describe svc web-service

#### What to Verify

* **ClusterIP / NodePort assigned**
* **Correct port mapping**
* **Selector matches Pods**

---

### Endpoints Check

* kubectl get endpoints web-service

* If `ENDPOINTS = <none>`:
  * Service is NOT connected to any Pod

---

### Label Matching

* kubectl get pods --show-labels

Ensure:
* Service `selector` == Pod labels

---

## 3. Check Pod Status

* kubectl get pods

Common states:
* `Running` → OK
* `Pending` → scheduling issue
* `CrashLoopBackOff` → app crash
* `ImagePullBackOff` → image issue

---

### Describe Pod

* kubectl describe pod <pod-name>

Look for:
* Events (bottom section)
* Image pull errors
* Resource issues
* Liveness/readiness probe failures

---

## 4. Check Container Logs

* kubectl logs <pod-name>

Follow logs:
* kubectl logs -f <pod-name>

Previous container logs:
* kubectl logs <pod-name> --previous

---

## 5. Exec Into Pod

* kubectl exec -it <pod-name> -- /bin/sh

Useful for:
* Checking app processes
* Testing internal connectivity
* Verifying environment variables

---

## 6. Check Application Dependencies (e.g., DB)

Repeat the same checks:
* Service
* Endpoints
* Pod status
* Logs

Example:
* kubectl get svc db-service
* kubectl get pods -l app=db
* kubectl logs <db-pod>

---

## 7. Check DNS Resolution

From inside a Pod:
* kubectl exec -it <pod> -- nslookup db-service

Or:

* dig db-service.default.svc.cluster.local

If DNS fails:

* Check CoreDNS pods:
  kubectl get pods -n kube-system

---

## 8. Check Network Connectivity

From inside Pod:
* kubectl exec -it <pod> -- curl http://db-service

If fails:
* Check NetworkPolicies
* Check CNI plugin
* Verify ports

---

## 9. Check Resource Constraints

* kubectl describe pod <pod-name>

Look for:
* OOMKilled
* CPU throttling
* Memory limits exceeded

---

## 10. Check Probes

Liveness and Readiness probes can restart Pods.

* kubectl describe pod <pod-name>

Look for:
* Failed probe events
* Incorrect probe configuration

---

## 11. Check Events

* kubectl get events --sort-by=.metadata.creationTimestamp

Shows:
* Scheduling failures
* Image pull errors
* Node issues

---

## 12. Check Node Status

* kubectl get nodes
* kubectl describe node <node-name>

Look for:
* `NotReady` state
* Resource pressure
* Network issues

---

## Common Failure Scenarios

### Service Has No Endpoints

* Cause: label mismatch
* Fix: align selector and pod labels

---

### Pod CrashLoopBackOff

* Cause: application error
* Fix: check logs, fix config/env

---

### ImagePullBackOff

* Cause: wrong image / registry access
* Fix: correct image or credentials

---

### DNS Not Working

* Cause: CoreDNS issue
* Fix: check kube-system pods

---

### Cannot Reach Service

* Cause:
  * NetworkPolicy blocking
  * Wrong port
* Fix: verify connectivity and policies

---

## Debugging Workflow Summary

1. User request (curl)
2. Service (selectors, endpoints)
3. Pod (status, logs)
4. Container (runtime issues)
5. Dependencies (DB, APIs)
6. DNS
7. Network
8. Node

---

## Best Practices

* Always debug **layer by layer**
* Use `kubectl describe` for events
* Use labels consistently
* Implement proper **readiness/liveness probes**
* Centralize logging (e.g., EFK/Prometheus stack)

---

## Key Takeaways

* Start troubleshooting from the **outside**
* Validate Service → Pod → Container → Dependency chain
* Logs and events are your primary debugging tools
* Most issues are caused by:
  * Label mismatches
  * Misconfiguration
  * Networking problems
