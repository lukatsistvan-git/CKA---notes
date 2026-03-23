# Custom Resources (CRD)

## Overview

A **CustomResourceDefinition (CRD)** allows you to extend the Kubernetes API by creating your own resource types.

This means:
* you are not limited to built-in resources (Pod, Service, Deployment)
* you can define custom objects that behave like native Kubernetes resources

---

## Why Use CRDs?

CRDs are used to:
* extend Kubernetes with custom APIs
* model application-specific resources
* enable automation via controllers/operators
* store structured configuration in the cluster

---

## Core Concepts

### CustomResourceDefinition (CRD)

* defines a new resource type
* acts as a **schema** for custom objects

---

### Custom Resource (CR)

* an instance of a CRD
* represents actual data

---

### Controller / Operator

* watches Custom Resources
* implements business logic
* reconciles desired vs actual state

---

## CRD vs Built-in Resources

CRDs behave like native resources:
* stored in etcd
* accessible via kubectl
* support CRUD operations

Example:
- kubectl get crontabs

---

## Example: CRD Definition

This defines a new resource type called **CronTab**.

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              cronSpec:
                type: string
              image:
                type: string
              replicas:
                type: integer

---

## Example: Custom Resource (CR)

Instance of the CRD:

apiVersion: stable.example.com/v1
kind: CronTab
metadata:
  name: my-cron
spec:
  cronSpec: "*/5 * * * *"
  image: my-cron-image

---

## How It Works

1. Create CRD → extends API
2. Create CR → stores desired state
3. Controller watches CR → applies logic

---

## CRD Scope

Two types:
* Namespaced → scoped to a namespace
* Cluster → cluster-wide

Example:

scope: Namespaced

---

## Validation Schema

CRDs use **OpenAPI v3 schema**:
* validates Custom Resources
* ensures correct structure

Benefits:
* prevents invalid configs
* enforces required fields

---

## Versioning

CRDs support multiple versions:
* v1, v2, etc.
* one version marked as storage

Example:

versions:
- name: v1
  served: true
  storage: true

---

## Subresources (Advanced)

Optional features:
* status → separate status updates
* scale → integration with HPA

Example:

subresources:
  status: {}

---

## CRD + Controller = Operator

A CRD alone only defines data.

To add behavior:
* implement a **controller**
* or use an **Operator**

Operator:
* continuously reconciles state
* automates complex workflows

Examples:

* database operators
* monitoring operators

---

## Important Notes

* CRDs are stored in **etcd**
* no logic without controller
* API extension is dynamic (no restart needed)

---

## Best Practices

* define validation schemas
* version your CRDs
* use meaningful API groups
* avoid breaking changes
* implement controllers for automation

---

## Common Pitfalls

* creating CRD without controller
* missing validation schema
* breaking API changes
* unclear naming conventions

---

## Key Takeaways

* CRD = extend Kubernetes API
* CR = instance of custom resource
* Controller = logic layer
* Operator = advanced controller pattern
* Essential for platform engineering

---

## Useful Commands

List CRDs
- kubectl get crd

Describe CRD
- kubectl describe crd crontabs.stable.example.com

List Custom Resources
- kubectl get crontabs
