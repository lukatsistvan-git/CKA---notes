# Kustomize

## Overview

**Kustomize** is a declarative configuration management tool for Kubernetes that allows you to **customize YAML manifests without templating**.

It is built into `kubectl` and enables:
* Reusability of base configurations
* Environment-specific customization
* Declarative patching and transformations

> Kustomize is part of the Kubernetes ecosystem and is officially supported.

---

## Core Concepts

### No Templating

* No `{{ }}` syntax (unlike Helm)
* Works by **patching and transforming YAML**

---

### kustomization.yaml

The central configuration file.

Defines:
* **resources** → base manifests
* **transformations / patches** → modifications

---

### Basic Example

resources:
  - deployment.yaml
  - service.yaml

patchesStrategicMerge:
  - patch.yaml

---

## Usage

### Build Manifests

kustomize build k8s/

* Generates combined YAML
* Does NOT apply to cluster

---

### Apply to Cluster

kustomize build k8s/ | kubectl apply -f -

Or using built-in support:

kubectl apply -k k8s/

---

### Delete Resources

kustomize build k8s/ | kubectl delete -f -

---

## File Structure

### Typical Layout

project-root/
├── kustomization.yaml
├── deployment.yaml
├── service.yaml

---

### Modular Structure

project-root/
├── kustomization.yaml
├── api/
│   ├── deployment.yaml
│   └── kustomization.yaml
├── db/
│   ├── statefulset.yaml
│   └── kustomization.yaml

---

### Root kustomization.yaml

resources:
  - api/
  - db/

* Recursively processes subdirectories

---

## Transformers

Transformers apply changes across multiple resources.

### Common Transformers

namePrefix: dev-
nameSuffix: -v1
namespace: dev
commonLabels:
  app: my-app
commonAnnotations:
  managed-by: kustomize


---

### Images

images:
  - name: nginx
    newName: myregistry/nginx
    newTag: "1.25.2"

---

## Patches

Kustomize supports two patch types:

---

### 1. Strategic Merge Patch (SMP)

* YAML-based
* Matches objects by **name + kind**
* Merges fields intelligently

#### Example

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: nginx:1.25.3

---

#### Delete Example

containers:
  - name: nginx
    $patch: delete

---

#### Usage

patchesStrategicMerge:
  - patch.yaml

---

### 2. JSON 6902 Patch

* Operation-based (precise control)
* Uses `add`, `replace`, `remove`

#### Example

patchesJson6902:
  - target:
      kind: Service
      name: my-service
    patch: |
      - op: replace
        path: /spec/type
        value: NodePort

---

### List Modification Example

- op: add
  path: /spec/template/spec/containers/0/env/-
  value:
    name: LOG_LEVEL
    value: debug

---

### External Patch File

patchesJson6902:
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: nginx
    path: patch.yaml

---

## Overlays

Overlays allow **environment-specific customization**.

---

### Structure

.
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch.yaml
    └── prod/
        ├── kustomization.yaml
        └── patch.yaml

---

### Base Example

# base/kustomization.yaml
resources:
  - deployment.yaml
  - service.yaml

---

### Overlay Example

# overlays/dev/kustomization.yaml
resources:
  - ../../base
patches:
  - path: patch.yaml
    target:
      kind: Deployment
      name: my-deployment

---

### Apply Overlay

kubectl apply -k overlays/dev/

---

### Benefits

* Reuse base configuration
* Customize per environment
* Clean separation of concerns

---

## Components

Components are **reusable configuration units**.

* Not full environments
* Optional features
* Applied via overlays

---

### Use Cases

* Debug sidecar
* Monitoring config
* Logging setup
* Extra environment variables

---

### Structure

components/
└── debug/
    ├── patch.yaml
    └── kustomization.yaml

---

### Component Example

resources: []
patches:
  - path: patch.yaml
    target:
      kind: Deployment
      name: my-app

---

### Using Components

# overlays/dev/kustomization.yaml
resources:
  - ../../base
components:
  - ../../components/debug

---

### Notes

* Cannot be applied standalone
* Only usable via `components:` field
* Multiple components can be combined

---

## Important Notes

* `apiVersion` and `kind` should always be defined explicitly
* Kustomize does NOT manage application lifecycle (only manifests)
* Works natively with `kubectl`

---

## Common Pitfalls

* Mixing Helm and Kustomize concepts
* Overusing patches instead of transformers
* Poor directory structure
* Hardcoding environment-specific values in base

---

## Best Practices

* Keep **base minimal and reusable**
* Use overlays for environment differences
* Prefer transformers over patches when possible
* Use clear directory structure (base / overlays / components)
* Version control all configurations

---

## Key Takeaways

* Kustomize = template-free Kubernetes configuration management
* Uses overlays and patches instead of templating
* Built into kubectl (`-k`)
* Ideal for multi-environment deployments
* Promotes clean, reusable configuration design
