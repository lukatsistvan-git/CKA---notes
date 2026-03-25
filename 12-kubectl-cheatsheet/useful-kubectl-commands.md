# Useful kubectl Commands (CKA Cheat Sheet)

## Overview

`kubectl` is the primary CLI tool for interacting with a Kubernetes cluster.

This cheat sheet focuses on **commonly used commands for CKA exam and real-world troubleshooting**.

---

## Cluster Information

kubectl cluster-info  
kubectl version  
kubectl config view  
kubectl config get-contexts  
kubectl config use-context <context>  

---

## Working with Nodes

kubectl get nodes  
kubectl describe node <node-name>  
kubectl top node  

---

## Working with Pods

kubectl get pods  
kubectl get pods -o wide  
kubectl describe pod <pod-name>  
kubectl delete pod <pod-name>  
kubectl logs <pod-name>  
kubectl logs -f <pod-name>  
kubectl logs <pod-name> --previous  
kubectl exec -it <pod-name> -- /bin/sh  

---

## Working with Deployments

kubectl get deployments  
kubectl describe deployment <name>  
kubectl create deployment <name> --image=<image>  
kubectl scale deployment <name> --replicas=3  
kubectl rollout status deployment <name>  
kubectl rollout history deployment <name>  
kubectl rollout undo deployment <name>  

---

## Working with Services

kubectl get svc  
kubectl describe svc <name>  
kubectl expose pod <pod-name> --port=80 --target-port=80 --type=ClusterIP  

---

## Namespaces

kubectl get ns  
kubectl create ns <name>  
kubectl delete ns <name>  
kubectl config set-context --current --namespace=<name>  

---

## Labels and Selectors

kubectl get pods --show-labels  
kubectl label pod <pod-name> key=value  
kubectl get pods -l key=value  

---

## Annotations

kubectl annotate pod <pod-name> key=value  

---

## Editing Resources

kubectl edit pod <pod-name>  
kubectl edit deployment <name>  

---

## Applying and Managing YAML

kubectl apply -f file.yaml  
kubectl apply -f directory/  
kubectl delete -f file.yaml  

---

## Output Formatting

kubectl get pods -o wide  
kubectl get pods -o yaml  
kubectl get pods -o json  

---

## Debugging & Troubleshooting

kubectl describe pod <pod-name>  
kubectl logs <pod-name>  
kubectl logs -f <pod-name>  
kubectl exec -it <pod-name> -- /bin/sh  
kubectl get events --sort-by=.metadata.creationTimestamp  

---

## Resource Usage

kubectl top pod  
kubectl top node  

---

## Port Forwarding

kubectl port-forward pod/<pod-name> 8080:80  

---

## File Copy

kubectl cp <pod-name>:/path/to/file /local/path  
kubectl cp /local/path <pod-name>:/path/to/file  

---

## ConfigMaps & Secrets

kubectl get configmap  
kubectl get secret  
kubectl describe configmap <name>  
kubectl describe secret <name>  

---

## Imperative Commands (CKA Important)

kubectl run nginx --image=nginx  
kubectl create deployment nginx --image=nginx  
kubectl expose deployment nginx --port=80 --type=ClusterIP  

---

## Useful Flags

-n <namespace>  
--all-namespaces  
-o wide  
--dry-run=client -o yaml  

---

## Aliases (Optional but Useful)

alias k=kubectl  

---

## Key Takeaways

* `kubectl get`, `describe`, `logs`, `exec` are core troubleshooting tools  
* Use `-o wide` and `-o yaml` for deeper inspection  
* Prefer `apply` over `create` for declarative workflows  
* Combine commands with labels and namespaces for precision  
* Practice imperative commands for CKA speed
