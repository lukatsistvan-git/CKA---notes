# Random Notes (CKA Quick Reference)

## Overview

This section contains **miscellaneous but highly useful Kubernetes notes** for:
* Exam preparation (CKA)
* Real-world troubleshooting
* Faster YAML and CLI usage

---

## Generate YAML Without Creating Resources

* kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-pod-def.yaml

### Explanation

* `--dry-run=client`
  * Runs locally (client-side only)
  * Does NOT create the resource in the cluster
  * Useful for generating manifests

* `-o yaml`
  * Outputs the resource definition in YAML format

---

## Imperative → YAML Workflow

Create Pod YAML:
* kubectl run nginx --image=nginx --dry-run=client -o yaml

Create Deployment YAML:
* kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

Save to file:
* kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml

Create from file:
* kubectl create -f nginx-deployment.yaml

Create with replicas:
* kubectl create deployment nginx --image=nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

---

## Editing Existing Resources (Recreate Pattern)

If `kubectl edit` is not enough:
1. kubectl get pod webapp -o yaml > my-new-pod.yaml
2. vi my-new-pod.yaml
3. kubectl delete pod webapp
4. kubectl create -f my-new-pod.yaml  

---

## kubectl explain (VERY IMPORTANT)

* kubectl explain pod  
* kubectl explain pod.spec  
* kubectl explain pod.spec.containers  

Use it to:
* Understand YAML structure
* Discover fields and types

---

## API Resources & Short Names

* kubectl api-resources  

Common short names:
* po → pod  
* svc → service  
* rs → replicaset  
* deploy → deployment  
* ns → namespace  

---

## Multiple Resources in One YAML

You can define multiple resources in a single YAML file using:

---

Example:

apiVersion: v1  
kind: Pod  
...  
---  
apiVersion: v1  
kind: Service  
...  

---

## Custom Output Formatting

* kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"

Useful for:
* Filtering specific fields
* Quick inspection

---

## Logs (Multiple Containers)

* kubectl logs <pod-name>  
* kubectl logs <pod-name> -c <container-name>  

---

## Default Namespace सेट (Set Default Namespace)

* kubectl config set-context --current --namespace=<namespaceName>

---

## SSH to Node

Get node IP:
* kubectl get nodes -o wide  

SSH:
* ssh <node-ip>  

---

## Static Pod Path

Location:
/var/lib/kubelet/config.yaml  

Look for:
staticPodPath  

Control Plane manifests typically in:
/etc/kubernetes/manifests  

---

## Admission Controllers (API Server)

List enabled admission plugins:
* ps -ef | grep kube-apiserver | grep admission-plugins  

---

## Certificates Inspection

Check certificate details:
* openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout  

Look for:
* Expiration date
* Common Name (CN)
* Issuer

---

## Useful Links

kubectl documentation:  
https://kubectl.docs.kubernetes.io/  

CKA course notes:  
https://github.com/kodekloudhub/certified-kubernetes-administrator-course/tree/master/docs
