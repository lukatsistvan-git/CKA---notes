# Helm

## Overview

**Helm** is the package manager for Kubernetes. It simplifies the deployment, upgrade, and management of applications using reusable, templated configurations called **Charts**.

Helm enables:
* Declarative application deployment
* Versioning and rollback
* Reusability and standardization
* Parameterized configuration via values

---

## Core Concepts (Helm 3)

### Chart

A **Chart** is a collection of files that describe a Kubernetes application.

* Contains:
  * YAML manifests
  * Templates
  * Default configuration (`values.yaml`)

---

### Release

A **Release** is a deployed instance of a Chart in a cluster.

* One Chart → multiple Releases possible
* Each release has its own configuration

---

### Revision

* Each **upgrade or rollback** creates a new revision
* Represents a snapshot of the release state

---

### Tiller (Removed)

* Helm 2 used **Tiller** (server-side component)
* Helm 3 removed it → fully **client-side architecture**

---

### 3-Way Strategic Merge Patch

Used during `helm upgrade`:

Helm compares:
1. Last deployed configuration
2. Current live state
3. New desired configuration

This ensures:
* Safer upgrades
* Minimal unintended changes

---

### Storage (Secrets)

Helm stores release metadata in:
* **Kubernetes Secrets** (default)
* Optionally ConfigMaps

Contains:
* Release history
* Values
* Manifest snapshots

---

### values.yaml

Defines configurable parameters for a Chart.

* Used in templates via placeholders
* Can be overridden at install/upgrade time

---

## Helm Chart Structure

my-chart/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
└── templates/_helpers.tpl

---

## Chart.yaml

Defines metadata about the Chart.

### Required Fields

apiVersion: v2
name: my-chart
version: 1.2.3

* `apiVersion`
  * Must be `v2` for Helm 3

* `name`
  * Chart name

* `version`
  * Chart version (SemVer)

---

### Optional Fields

description: A Helm chart for Kubernetes
type: application
appVersion: "1.0.0"
keywords:
  - web
maintainers:
  - name: Jane Doe
    email: jane@example.com
sources:
  - https://github.com/myorg/myapp
home: https://myapp.example.com

---

### Dependencies

dependencies:
  - name: redis
    version: 17.3.11
    repository: https://charts.bitnami.com/bitnami

* Defines **subcharts**
* Managed via `helm dependency update`

---

## Chart Repositories

### Add Repository

helm repo add bitnami https://charts.bitnami.com/bitnami

---

### Update Repositories

helm repo update

---

### List Repositories

helm repo list

---

### Remove Repository

helm repo remove bitnami

---

## Searching Charts

### Search Local Repositories

helm search repo wordpress

---

### Search Artifact Hub

helm search hub wordpress

---

## Installing Charts

### Basic Install

helm install my-release bitnami/wordpress

---

### Override Values (CLI)

helm install my-release bitnami/wordpress --set image.tag=latest

---

### Use values.yaml File

helm install my-release bitnami/wordpress -f custom-values.yaml

---

### Multiple Values Files

helm install my-release bitnami/wordpress -f base.yaml -f prod.yaml

---

## Working with Local Charts

### Download Chart

helm pull bitnami/wordpress

---

### Download and Extract

helm pull --untar bitnami/wordpress

---

### Install Local Chart

helm install my-release ./wordpress

---

## Managing Releases

### List Releases

helm list
helm list --namespace my-namespace
helm list --all-namespaces

---

### Release History

helm history my-release

Shows:
* Revision number
* Timestamp
* Status
* Description

---

### Rollback

helm rollback my-release 1

* Rolls back to revision 1
* Creates a **new revision**

---

### Upgrade

helm upgrade my-release bitnami/wordpress

With values:
helm upgrade my-release bitnami/wordpress -f values.yaml

---

### Uninstall

helm uninstall my-release

* Deletes:
  * All Kubernetes resources created by the Chart
  * Release metadata

---

## Templating Basics

Helm uses **Go templates**.

Example:
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}

---

## Common Pitfalls

* Hardcoding values instead of using `values.yaml`
* Forgetting to run `helm repo update`
* Not tracking revisions before upgrade
* Misusing `--set` for complex configs
* Ignoring dependency updates

---

## Best Practices

* Use **values.yaml** for configuration
* Keep Charts **modular and reusable**
* Version Charts properly (SemVer)
* Use **helm lint** to validate charts
* Store Charts in repositories (Artifact Hub, private repos)
* Avoid large `--set` overrides → prefer files

---

## Key Takeaways

* Helm simplifies Kubernetes application management
* Charts = reusable deployment templates
* Releases = running instances of Charts
* Built-in versioning and rollback
* Strong ecosystem and tooling support
