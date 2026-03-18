# Application Logs

## Overview

In Kubernetes, application logging is based on container standard output streams:
- stdout (standard output)
- stderr (standard error)

The container runtime (e.g., containerd, CRI-O) captures these logs and stores them on the node.

Kubernetes itself does not provide built-in log storage or aggregation.

---

## How Logging Works in Kubernetes

- Applications write logs to stdout/stderr
- The container runtime collects logs
- Logs are stored on the node filesystem
- kubectl logs retrieves and displays them

Important:
- Logs are tied to the lifecycle of the container
- If a Pod is deleted, logs may be lost (unless centralized logging is used)

---

## Log Storage

Container logs are stored on the node, typically under:
- /var/log/containers/
- /var/log/pods/

Key points:
- Managed by the container runtime
- Rotated automatically
- Not retained long-term
- Lost if node is destroyed

---

## Limitations of Native Logging

- No centralized storage
- No long-term retention
- No advanced querying
- Hard to debug across multiple Pods/nodes

Because of this, production systems use centralized logging solutions.

---

## Centralized Logging Solutions

Common logging stack components:

### Log Collectors
- Fluentd
- Logstash
- Fluent Bit

These:
- Run as DaemonSets
- Collect logs from nodes
- Forward logs to storage backends

### Storage & Search
- Elasticsearch
- Loki

Provide:
- Indexing
- Search capabilities
- Log retention

### Visualization
- Grafana
- Kibana

Used for:
- Dashboards
- Log exploration
- Troubleshooting

---

## Logging Patterns

### Node-Level Logging (Recommended)
- Logs are collected at node level
- Usually via DaemonSet (e.g., Fluentd)
- Works for all containers automatically

### Sidecar Logging
- A sidecar container processes logs

Example:
- Main container writes logs to file
- Sidecar reads and forwards logs

Use case:
- Custom log formats
- Legacy applications

---

## Troubleshooting with Logs

Logs are often the first step in debugging.

Common scenarios:
- Pod crashes → check --previous logs
- Application errors → inspect stdout
- Multi-container issues → check each container separately
- Startup issues → combine logs + kubectl describe pod

---

## Debugging Workflow (Recommended)

A typical investigation flow:

Check Pod status
- kubectl get pods

Describe the Pod
- kubectl describe pod pod-name

Check logs
- kubectl logs pod-name

If restarted
- kubectl logs pod-name --previous

If multi-container
- kubectl logs pod-name -c container-name

If still unclear
- Check events (describe output)
- Check node status

---

## Pro Tips

- Always start with kubectl logs + describe
- Use --previous for crashes → critical for CKA
- Logs + Events together give full picture
- Not all issues appear in logs (e.g., scheduling problems)
- Centralized logging is mandatory in production

---

## Key Takeaways

- Kubernetes uses stdout/stderr for logging
- kubectl logs is the primary debugging tool
- Logs are stored locally on nodes
- Logs are ephemeral by default
- Centralized logging is essential in production
- --previous is critical for crash debugging

---

## Useful Commands

Get logs from a Pod:
- kubectl logs pod-name

Follow logs:
- kubectl logs -f pod-name

Logs from specific container:
- kubectl logs pod-name -c container-name

Previous logs (after restart):
- kubectl logs pod-name --previous

All containers:
- kubectl logs pod-name --all-containers=true
