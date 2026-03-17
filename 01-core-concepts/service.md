# Services

## What is a Service

A **Service** is a Kubernetes object that provides a **stable network endpoint** for a set of Pods.
A Service selects Pods using **labels and selectors**.

Pods are **ephemeral**:
- their IP addresses change
- they can be created or deleted at any time

A Service solves this by providing:
- a **stable IP address**
- a **DNS name**
- **load balancing** across Pods

---

## What a Service Does

A Service can:
- expose an application to other Pods or external users
- provide **service discovery** via DNS
- distribute traffic between Pods (**load balancing**)
- decouple clients from Pods

---

## Types of Services

There are three main Service types.

1. ClusterIP (Default)

The Service is only accessible inside the cluster.

Use case:
- frontend → backend communication
- backend → database communication

Characteristics:
- internal IP only
- accessible via ClusterIP or DNS name
- not exposed externally

Example:
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80

2. NodePort

Exposes the Service externally using a port on each Node.

Traffic flow:
- Client → NodeIP:NodePort → Service → Pod

Characteristics:
- accessible from outside the cluster
- uses a port range: 30000–32767
- works on all worker nodes

Important ports:
- nodePort: external port on the Node
- port: Service port
- targetPort: container port inside the Pod

Example:
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007

3. LoadBalancer

Exposes the Service externally using a cloud provider load balancer.

Characteristics:
- provides an external IP address
- integrates with cloud providers (AWS, GCP, Azure)
- automatically provisions a load balancer

Example:
apiVersion: v1
kind: Service
metadata:
  name: myapp-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80

Note:
- works automatically only in cloud environments
- on local clusters (e.g. Minikube), additional setup is required

---

## Service Discovery (DNS)

Each Service gets a DNS name inside the cluster.

Format:
- <service-name>.<namespace>.svc.cluster.local

Example:
- myapp-service.default.svc.cluster.local
- myapp-service (short form, in case of default namespace, default settings)

---

## Load Balancing

Services distribute traffic across Pods.

Default behavior:
- simple load balancing (round-robin)
- each request goes to one Pod
- traffic is distributed evenly

If a Pod becomes unavailable:
- it is automatically removed from the endpoints
- traffic is redirected to healthy Pods

---

## Troubleshooting
Service not working?

Check:
Pod labels:
- kubectl get pods --show-labels
Service selector:
- kubectl describe service myapp-service
Endpoints:
- kubectl get endpoints
If no endpoints are listed → selector does not match Pods.
Test connectivity inside cluster
- kubectl run tmp-shell --rm -it --image=busybox -- sh
Then:
- wget -qO- http://myapp-service
or
- nslookup myapp-service / nslookup <service-name>.<namespace>

---

## Key Takeaways

- ervices provide a stable network endpoint
- they use labels and selectors to target Pods
- they enable service discovery via DNS
- they provide built-in load balancing
- three main types:
-- ClusterIP (internal)
-- NodePort (external via node)
-- LoadBalancer (external via cloud)

---

## Useful Commands

Create Service:
- kubectl create -f service-definition.yaml 
- kubectl apply -f service-definition.yaml
- kubectl create service [flags]
or we can expose a resource (pod, service, deployment, replicaset) as a new Kubernetes service:
- kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP|SCTP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]

List Services
- kubectl get svc

Describe Service
- kubectl describe svc myapp-service

Check endpoints
- kubectl get endpoints

Delete Service
- kubectl delete svc myapp-service
