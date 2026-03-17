# Multiple Schedulers

## Overview

Kubernetes allows running **multiple schedulers** in the same cluster.
- default scheduler: `kube-scheduler`
- custom schedulers: user-defined

Each Pod can choose which scheduler to use.

---

## Why Use Multiple Schedulers?

- different scheduling strategies
- special workloads
- custom logic (e.g. GPU, batch jobs)
- isolation between workloads

---

## Core Concept

Each Pod specifies:
spec:
  schedulerName: my-custom-scheduler

If not specified:
- default scheduler (kube-scheduler) is used

---

## How It Works

- Multiple scheduler instances run in the cluster
- Each scheduler watches for Pods assigned to its name
- Scheduler only handles matching Pods

---

## Creating a Custom Scheduler

Example Pod:

apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - name: kube-scheduler
    image: registry.k8s.io/kube-scheduler:v1.29.0
    command:
      - kube-scheduler
      - --config=/etc/kubernetes/my-scheduler-config.yaml
    volumeMounts:
    - name: scheduler-config
      mountPath: /etc/kubernetes
  volumes:
  - name: scheduler-config
    configMap:
      name: my-scheduler-config

---

## Using a Custom Scheduler

Example Pod:

apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  schedulerName: my-custom-scheduler
  containers:
  - name: nginx
    image: nginx

---

## Important Notes

- Scheduler version should match cluster version
- Scheduler requires a valid kubeconfig (scheduler.conf)
- Default scheduler continues to run unless disabled

---

## Troubleshooting

Pod not scheduled
- kubectl describe pod <pod-name>

Check:
- schedulerName mismatch
- no scheduler running with that name

Check schedulers
- kubectl get pods -n kube-system

Check events
- kubectl get events -o wide

Check logs
- kubectl logs my-custom-scheduler -n kube-system

---

## Key Takeaways

- Kubernetes supports multiple schedulers
- Pods select scheduler via schedulerName
- Custom schedulers enable flexible scheduling logic
- Default scheduler remains active unless explicitly replaced
