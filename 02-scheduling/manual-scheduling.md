# Manual Scheduling

## What is Manual Scheduling

**Manual scheduling** means assigning a Pod to a specific node **without using the kube-scheduler**.

Instead of Kubernetes deciding where to place the Pod, you explicitly define the target node.

---

## When to Use Manual Scheduling

Manual scheduling is rarely used in production, but useful in:
- learning and testing environments
- debugging scheduling issues
- special cases where a Pod must run on a specific node
- environments without a scheduler

---

## How Manual Scheduling Works

To manually schedule a Pod, use the `nodeName` field in the Pod specification.

Example:

apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: worker-node-1

---

## What Happens Internally

Normally:
- Pod → Scheduler → Node assignment

With manual scheduling:
- Pod → nodeName set → directly assigned to node

Important:
- The Pod bypasses the scheduler completely.
- The kubelet on the specified node receives the Pod and starts it.

---

## Important Notes

- If the specified node does not exist, the Pod will remain in Pending state
- If the node is not ready, the Pod will not start
- No scheduling logic is applied (resources, affinity, etc. are ignored)

---

## nodeName vs nodeSelector

This is a very important distinction.

nodeName
- directly assigns a Pod to a node
- bypasses scheduler
- no validation or scheduling logic

nodeSelector
- uses labels to select nodes
- still uses the scheduler
- allows flexible placement

---

## Why Manual Scheduling is Not Recommended?

Manual scheduling has several limitations:
- not scalable
- no resource awareness
- no failover handling
- tightly coupled to specific nodes

Because of this, it is NOT recommended in production environments.

---

## Key Takeaways

- Manual scheduling uses the nodeName field
- It bypasses the kube-scheduler
- It directly assigns Pods to nodes
- It is mainly used for testing and debugging
- Not recommended for production use
