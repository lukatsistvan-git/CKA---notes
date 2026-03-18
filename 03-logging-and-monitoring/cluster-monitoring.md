# Cluster monitoring

## Overview

Kubernetes does not include a full-featured, built-in monitoring solution.
Instead, it relies on an ecosystem of tools that can be integrated to collect, store, visualize, and alert on cluster metrics.

Monitoring typically covers:
- Node resource usage (CPU, memory)
- Pod resource usage
- Application metrics
- Cluster object states
- Events and failures

---

## Metrics Server

Metrics Server is a lightweight cluster-wide aggregator of resource usage data.

It collects metrics from:
- Kubelet on each node

Provides:
- CPU usage
- Memory usage

Key characteristics:
- Stores data in-memory only
- Not suitable for long-term monitoring
- Used for real-time insights

Required for:
- Horizontal Pod Autoscaler (HPA)
- kubectl top commands

---

## Prometheus

Prometheus is the de facto standard monitoring system for Kubernetes.

Key features:
- Pull-based metrics collection
- Time-series database
- Powerful query language (PromQL)

How it works:
- Scrapes metrics from endpoints (e.g., /metrics)
- Stores data over time
- Enables querying and alerting

Common use cases:
- Cluster monitoring
- Application metrics
- Performance analysis

---

## Grafana

Grafana is used for visualization of metrics.

Features:
- Dashboards with graphs and charts
- Integration with Prometheus
- Custom queries and panels

Benefits:
- User-friendly interface
- Highly customizable
- Widely used in production environments

---

## Alertmanager

Alertmanager works with Prometheus to handle alerts.

Responsibilities:
- Deduplicating alerts
- Grouping alerts
- Routing alerts to receivers

Supported notifications:
- Email
- Slack
- Webhooks
- PagerDuty

Example use cases:
- High CPU usage
- Pod crash loops
- Node not ready

---

## kube-state-metrics

kube-state-metrics exposes metrics about Kubernetes objects.

It does NOT monitor resource usage directly.

Instead, it provides state information about:
- Deployments
- ReplicaSets
- Pods
- Nodes
- DaemonSets

Example insights:
- Desired vs available replicas
- Pod status (Running, Pending, Failed)

Used together with:
- Prometheus (for scraping)
- Grafana (for visualization)

---

## Monitoring Architecture (Typical Stack)

A common Kubernetes monitoring stack:
- Metrics Server → real-time resource usage
- Prometheus → metrics collection & storage
- kube-state-metrics → cluster object state
- Alertmanager → alert handling
- Grafana → visualization

---

## Key Takeaways

- Kubernetes has no full built-in monitoring solution
- Metrics Server provides basic, real-time metrics
- Prometheus is the standard monitoring system
- Grafana is used for visualization
- Alertmanager handles alerts and notifications
- kube-state-metrics provides cluster state metrics

---

## Useful Commands

These commands work only if Metrics Server is installed.

Check node resource usage:
- kubectl top nodes

Show metrics for all pods in the default namespace:
- kubectl top pods

Show metrics for all pods in the given namespace
- kubectl top pod --namespace=NAMESPACE
  
Show metrics for a given pod and its containers
- kubectl top pod POD_NAME --containers
  
Show metrics for the pods defined by label name=myLabel
- kubectl top pod -l name=myLabel

Example output includes:
- CPU (cores or millicores)
- Memory (Mi/Gi)
