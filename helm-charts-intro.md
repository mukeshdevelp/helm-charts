# Helm Charts — Questions and Answers

This README contains commonly asked **Helm and Helm Charts questions with answers**, starting from basic concepts and moving toward practical and interview-level topics.

---

## Table of Contents

* [1. What is Helm?](#1-what-is-helm)
* [2. Why use Helm?](#2-why-use-helm)
* [3. What is a Helm Chart?](#3-what-is-a-helm-chart)
* [4. Helm vs Kubernetes](#4-helm-vs-kubernetes)
* [5. What is Helm 3?](#5-what-is-helm-3)
* [6. What is a Helm Release?](#6-what-is-a-helm-release)
* [7. Chart vs Release](#7-chart-vs-release)
* [8. Helm Chart Structure](#8-helm-chart-structure)
* [9. Chart.yaml](#9-chartyaml)
* [10. values.yaml](#10-valuesyaml)
* [11. templates Directory](#11-templates-directory)
* [12. _helpers.tpl](#12-helperstpl)
* [13. charts Directory](#13-charts-directory)
* [14. .helmignore](#14-helmignore)
* [15. Helm Templating](#15-helm-templating)
* [16. Accessing values.yaml](#16-accessing-valuesyaml)
* [17. --set vs -f](#17---set-vs--f)
* [18. Multiple Values Files](#18-multiple-values-files)
* [19. Helm Repository](#19-helm-repository)
* [20. Installing a Chart](#20-installing-a-chart)
* [21. Listing Releases](#21-listing-releases)
* [22. Checking Release Status](#22-checking-release-status)
* [23. Viewing Release Values](#23-viewing-release-values)
* [24. Viewing Generated Manifests](#24-viewing-generated-manifests)
* [25. helm template](#25-helm-template)
* [26. helm lint](#26-helm-lint)
* [27. Upgrading a Release](#27-upgrading-a-release)
* [28. Rollback](#28-rollback)
* [29. Release History](#29-release-history)
* [30. Troubleshooting Helm](#30-troubleshooting-helm)
* [31. Environment-specific Deployments](#31-environment-specific-deployments)
* [32. Helm Dependencies](#32-helm-dependencies)
* [33. Helm Hooks](#33-helm-hooks)
* [34. Important Helm Commands](#34-important-helm-commands)

---

# 1. What is Helm?

**Helm is a package manager for Kubernetes.**

It allows us to package Kubernetes resources such as:

* Deployment
* Service
* ConfigMap
* Secret
* Ingress
* ServiceAccount
* PersistentVolume
* PersistentVolumeClaim

into a reusable package called a **Helm Chart**.

Instead of maintaining many Kubernetes YAML files manually, Helm allows us to create templates and provide configuration through values.

Example:

```bash
helm install myapp ./mychart
```

---

# 2. Why use Helm?

Without Helm, we might have:

```text
deployment.yaml
service.yaml
configmap.yaml
secret.yaml
ingress.yaml
```

For different environments, we may need different versions of these files.

Helm allows us to create reusable templates:

```text
templates/
    deployment.yaml
    service.yaml
    configmap.yaml
    ingress.yaml
```

and configure them using:

```text
values.yaml
```

This makes Kubernetes deployments:

* Reusable
* Configurable
* Versioned
* Easier to upgrade
* Easier to rollback
* Easier to manage across environments

---

# 3. What is a Helm Chart?

A **Helm Chart** is a collection of files that describes a Kubernetes application.

A chart normally contains:

```text
Chart.yaml
values.yaml
templates/
charts/
```

For example:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── ingress.yaml
└── charts/
```

---

# 4. Helm vs Kubernetes

Kubernetes is the **container orchestration platform**.

Helm is a **package manager and templating tool for Kubernetes**.

For example:

```text
Kubernetes
    ↓
Runs containers and manages resources

Helm
    ↓
Packages and deploys Kubernetes resources
```

Helm does not replace Kubernetes.

Helm generates and manages Kubernetes resources, while Kubernetes actually runs those resources.

---

# 5. What is Helm 3?

Helm 3 is the modern version of Helm.

One major difference from Helm 2 is that **Helm 3 removed Tiller**.

Helm 2 used:

```text
Helm Client
     ↓
Tiller
     ↓
Kubernetes
```

Helm 3 works directly with the Kubernetes API:

```text
Helm
  ↓
Kubernetes API
  ↓
Kubernetes Resources
```

---

# 6. What is a Helm Release?

A **Release** is an installed instance of a Helm Chart.

For example:

```bash
helm install myapp ./mychart
```

Here:

```text
mychart = Chart
myapp   = Release
```

The same chart can be installed multiple times:

```bash
helm install dev-app ./mychart
helm install staging-app ./mychart
helm install prod-app ./mychart
```

All three are different releases of the same chart.

---

# 7. Chart vs Release

### Chart

A chart is the package/template.

```text
mychart/
├── Chart.yaml
├── values.yaml
└── templates/
```

### Release

A release is a deployed instance of that chart.

```text
mychart
   ↓
helm install
   ↓
myapp
   ↓
Release
```

Therefore:

```text
Chart = Package

Release = Installed instance of the package
```

---

# 8. Helm Chart Structure

A typical Helm chart looks like:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── _helpers.tpl
└── .helmignore
```

---

# 9. Chart.yaml

`Chart.yaml` contains metadata about the chart.

Example:

```yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
type: application
version: 0.1.0
appVersion: "1.0.0"
```

Important fields:

### `apiVersion`

Specifies the Helm chart API version.

For Helm 3 charts, this is normally:

```yaml
apiVersion: v2
```

### `name`

Chart name.

```yaml
name: myapp
```

### `version`

Version of the Helm chart.

```yaml
version: 0.1.0
```

### `appVersion`

Version of the application being deployed.

```yaml
appVersion: "1.0.0"
```

---

# 10. values.yaml

`values.yaml` contains default configuration values for the chart.

Example:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```

Templates can read these values.

For example:

```yaml
replicas: {{ .Values.replicaCount }}
```

---

# 11. templates Directory

The `templates/` directory contains Kubernetes resource templates.

Example:

```text
templates/
├── deployment.yaml
├── service.yaml
├── configmap.yaml
├── secret.yaml
└── ingress.yaml
```

Helm processes these templates and generates Kubernetes YAML.

For example:

```yaml
replicas: {{ .Values.replicaCount }}
```

could become:

```yaml
replicas: 2
```

---

# 12. _helpers.tpl

`_helpers.tpl` is commonly used to define reusable template functions.

Example:

```yaml
{{- define "myapp.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end }}
```

Then it can be reused:

```yaml
name: {{ include "myapp.fullname" . }}
```

This prevents duplication in templates.

---

# 13. charts Directory

The `charts/` directory can contain chart dependencies.

For example:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── charts/
│   └── redis/
└── templates/
```

This allows one Helm chart to depend on another chart.

---

# 14. .helmignore

`.helmignore` specifies files that should not be included when packaging a chart.

It works similarly to:

```text
.gitignore
```

Example:

```text
.git/
.gitignore
README.md
*.tmp
```

---

# 15. Helm Templating

Helm uses the **Go template language**.

For example:

```yaml
replicas: {{ .Values.replicaCount }}
```

If `values.yaml` contains:

```yaml
replicaCount: 3
```

Helm generates:

```yaml
replicas: 3
```

Another example:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

With:

```yaml
image:
  repository: nginx
  tag: "1.27"
```

the result becomes:

```yaml
image: "nginx:1.27"
```

---

# 16. Accessing values.yaml

Values are accessed using:

```text
.Values
```

For example:

```yaml
replicaCount: 3
```

can be accessed with:

```yaml
{{ .Values.replicaCount }}
```

Nested values:

```yaml
image:
  repository: nginx
  tag: "1.27"
```

are accessed using:

```yaml
{{ .Values.image.repository }}
```

and:

```yaml
{{ .Values.image.tag }}
```

---

# 17. --set vs -f

There are two common ways to override Helm values.

### Using `--set`

```bash
helm install myapp ./mychart \
  --set replicaCount=3
```

This overrides:

```yaml
replicaCount: 2
```

with:

```yaml
replicaCount: 3
```

### Using `-f`

```bash
helm install myapp ./mychart -f production.yaml
```

This allows us to maintain environment-specific configuration in files.

For example:

```text
values.yaml
values-dev.yaml
values-prod.yaml
```

---

# 18. Multiple Values Files

Multiple values files can be supplied.

Example:

```bash
helm install myapp ./mychart \
  -f values.yaml \
  -f values-prod.yaml
```

The later values file overrides conflicting values from the earlier file.

For example:

```text
values.yaml
      ↓
values-prod.yaml
      ↓
Final values
```

`--set` can also override values from the files.

---

# 19. Helm Repository

A Helm repository stores Helm charts.

Add a repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Update repositories:

```bash
helm repo update
```

List repositories:

```bash
helm repo list
```

Search charts:

```bash
helm search repo nginx
```

Remove a repository:

```bash
helm repo remove bitnami
```

---

# 20. Installing a Chart

Install a local chart:

```bash
helm install myapp ./mychart
```

Install a chart from a repository:

```bash
helm install myapp bitnami/nginx
```

Specify a namespace:

```bash
helm install myapp ./mychart \
  --namespace production \
  --create-namespace
```

---

# 21. Listing Releases

List releases in the current namespace:

```bash
helm list
```

List releases in a specific namespace:

```bash
helm list -n production
```

List releases across all namespaces:

```bash
helm list -A
```

---

# 22. Checking Release Status

Use:

```bash
helm status myapp
```

For a specific namespace:

```bash
helm status myapp -n production
```

This provides information about:

* Release status
* Revision
* Namespace
* Resources
* Notes

---

# 23. Viewing Release Values

View values used by a release:

```bash
helm get values myapp
```

View all values, including defaults:

```bash
helm get values myapp --all
```

---

# 24. Viewing Generated Manifests

To see the Kubernetes resources generated for a release:

```bash
helm get manifest myapp
```

This is very useful when troubleshooting.

---

# 25. helm template

`helm template` renders the chart locally without installing it.

Example:

```bash
helm template myapp ./mychart
```

This allows us to inspect the generated Kubernetes YAML.

For example:

```bash
helm template myapp ./mychart > generated.yaml
```

Then inspect:

```bash
cat generated.yaml
```

### Difference

```text
helm template
      ↓
Render YAML only
      ↓
Does NOT deploy
```

Whereas:

```text
helm install
      ↓
Render templates
      ↓
Send resources to Kubernetes
      ↓
Deploy application
```

---

# 26. helm lint

`helm lint` checks a chart for potential problems.

Example:

```bash
helm lint ./mychart
```

It can identify issues such as:

* Invalid chart structure
* Template problems
* Incorrect metadata
* Some configuration errors

A good workflow is:

```bash
helm lint ./mychart
helm template myapp ./mychart
helm install myapp ./mychart
```

---

# 27. Upgrading a Release

To upgrade an existing release:

```bash
helm upgrade myapp ./mychart
```

For example, change:

```yaml
replicaCount: 2
```

to:

```yaml
replicaCount: 3
```

and run:

```bash
helm upgrade myapp ./mychart
```

Helm creates a new release revision.

---

# 28. Rollback

If an upgrade causes a problem, Helm can rollback to a previous revision.

First check history:

```bash
helm history myapp
```

Then rollback:

```bash
helm rollback myapp 1
```

Here:

```text
myapp = release name
1     = revision number
```

---

# 29. Release History

View release history:

```bash
helm history myapp
```

Example:

```text
REVISION    STATUS
1           deployed
2           superseded
3           deployed
```

Helm maintains release revisions, which makes rollback possible.

---

# 30. Troubleshooting Helm

When a Helm deployment fails, use the following commands.

### Check release status

```bash
helm status myapp
```

### Check history

```bash
helm history myapp
```

### Check generated YAML

```bash
helm template myapp ./mychart
```

### Check release values

```bash
helm get values myapp
```

### Check generated manifest

```bash
helm get manifest myapp
```

### Check Kubernetes resources

```bash
kubectl get pods
kubectl get deployments
kubectl get services
```

### Check Pod details

```bash
kubectl describe pod <pod-name>
```

### Check Pod logs

```bash
kubectl logs <pod-name>
```

A useful troubleshooting flow is:

```text
Helm deployment
      ↓
helm status
      ↓
helm get values
      ↓
helm get manifest
      ↓
kubectl get pods
      ↓
kubectl describe pod
      ↓
kubectl logs
```

---

# 31. Environment-specific Deployments

Helm is commonly used to deploy the same application to multiple environments.

Example:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
└── templates/
```

Development:

```bash
helm install myapp ./mychart \
  -f values-dev.yaml
```

Production:

```bash
helm install myapp ./mychart \
  -f values-prod.yaml
```

For example:

### values-dev.yaml

```yaml
replicaCount: 1

image:
  tag: "dev"
```

### values-prod.yaml

```yaml
replicaCount: 3

image:
  tag: "1.0.0"
```

The same templates are reused while configuration changes between environments.

---

# 32. Helm Dependencies

A Helm chart can depend on other charts.

For example:

```text
My Application
      ↓
    Redis
      ↓
   PostgreSQL
```

Dependencies can be defined in `Chart.yaml`.

Example:

```yaml
dependencies:
  - name: redis
    version: "20.x.x"
    repository: "https://charts.bitnami.com/bitnami"
```

Then download dependencies:

```bash
helm dependency update
```

List dependencies:

```bash
helm dependency list
```

---

# 33. Helm Hooks

Helm hooks allow us to run Kubernetes resources at specific points during a Helm operation.

Examples include:

```text
pre-install
post-install
pre-upgrade
post-upgrade
pre-delete
post-delete
```

Example:

```yaml
metadata:
  annotations:
    "helm.sh/hook": pre-install
```

Hooks can be useful for tasks such as:

* Database migrations
* Initialization
* Cleanup
* Post-deployment tasks

However, hooks should be used carefully because they can make deployments more complicated.

---

# 34. Important Helm Commands

## Repository commands

```bash
helm repo add
helm repo update
helm repo list
helm repo remove
helm search repo
```

## Chart commands

```bash
helm create mychart
helm lint ./mychart
helm package ./mychart
helm template myapp ./mychart
```

## Release commands

```bash
helm install
helm upgrade
helm upgrade --install
helm uninstall
helm list
helm status
helm history
helm rollback
```

## Information commands

```bash
helm get values
helm get manifest
helm get all
helm get notes
```

## Dependency commands

```bash
helm dependency update
helm dependency build
helm dependency list
```

---

# Important Interview Questions

The following questions are especially important to understand:

### 1. What is Helm?

Helm is a package manager for Kubernetes that uses charts to package, configure, deploy, and manage Kubernetes applications.

### 2. What is a Helm Chart?

A Helm Chart is a package containing Kubernetes resource templates and configuration.

### 3. What is a Helm Release?

A release is an installed instance of a Helm chart.

### 4. What is the difference between Chart and Release?

```text
Chart   → Package/template
Release → Deployed instance of the chart
```

### 5. What is values.yaml?

It contains the default configuration values used by Helm templates.

### 6. What is the templates directory?

It contains Kubernetes YAML templates that Helm processes before deployment.

### 7. What is `helm template`?

It renders the Helm templates into Kubernetes YAML without deploying them.

### 8. What is `helm lint`?

It validates a Helm chart for common errors and structural problems.

### 9. How do you upgrade a release?

```bash
helm upgrade myapp ./mychart
```

### 10. How do you rollback?

```bash
helm rollback myapp <revision>
```

### 11. How do you troubleshoot a Helm deployment?

Start with:

```bash
helm status
helm get values
helm get manifest
helm history
kubectl get pods
kubectl describe pod
kubectl logs
```

---

# Recommended Learning Order

Learn Helm in this order:

```text
1. Kubernetes YAML
        ↓
2. What is Helm?
        ↓
3. Helm Chart
        ↓
4. Chart.yaml
        ↓
5. values.yaml
        ↓
6. templates/
        ↓
7. Helm templating
        ↓
8. helm install
        ↓
9. helm upgrade
        ↓
10. helm rollback
        ↓
11. Environment-specific values
        ↓
12. Helm dependencies
        ↓
13. Helm hooks
        ↓
14. Reusable/production Helm charts
```

---

# Quick Revision

```text
Helm
 └── Package manager for Kubernetes

Chart
 └── Package containing Kubernetes templates

Release
 └── Installed instance of a Chart

Chart.yaml
 └── Chart metadata

values.yaml
 └── Default configuration

templates/
 └── Kubernetes resource templates

charts/
 └── Chart dependencies

_helpers.tpl
 └── Reusable template helpers

helm install
 └── Install a chart

helm upgrade
 └── Upgrade a release

helm rollback
 └── Roll back to an earlier revision

helm template
 └── Render YAML without deploying

helm lint
 └── Validate a chart

helm history
 └── Show release revisions

helm get manifest
 └── Show generated Kubernetes resources
```

---

# Practice Project

A good practical Helm project is to convert an existing Kubernetes application into a Helm chart.

Start with:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Then create:

```bash
helm create myapp
```

Move the Kubernetes resources into:

```text
myapp/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── configmap.yaml
```

Replace hard-coded values with:

```yaml
{{ .Values.someValue }}
```

Then test:

```bash
helm lint ./myapp

helm template myapp ./myapp

helm install myapp ./myapp

helm list

helm status myapp

helm upgrade myapp ./myapp

helm history myapp

helm rollback myapp 1
```

This gives you practical experience with the complete Helm lifecycle:

```text
Create
  ↓
Template
  ↓
Lint
  ↓
Install
  ↓
Inspect
  ↓
Upgrade
  ↓
Rollback
```

---

# Conclusion

Helm simplifies Kubernetes application management by providing:

* Reusable templates
* Configuration management
* Application packaging
* Versioning
* Release management
* Upgrade capabilities
* Rollback capabilities
* Dependency management
* Environment-specific configuration

The most important concepts to master are:

**Chart → Values → Templates → Release → Install → Upgrade → Rollback**
