# Creating a Helm Chart From Scratch

This guide explains how to create a **Helm Chart from scratch** and deploy an application to Kubernetes using Helm.

The goal is to understand:

* What a Helm Chart contains
* How to create the chart
* How `Chart.yaml` works
* How `values.yaml` works
* How Helm templates work
* How to create Deployment and Service templates
* How to render and validate templates
* How to install, upgrade, and rollback a Helm release

## Quick Navigation

## Table of Contents

| # | Section |
|---:|---|
| 1 | [Prerequisites](#1-prerequisites) |
| 2 | [What Are We Going to Build?](#2-what-are-we-going-to-build) |
| 3 | [Create the Project Directory](#3-create-the-project-directory) |
| 4 | [Create a Helm Chart](#4-create-a-helm-chart) |
| 5 | [Understanding the Generated Structure](#5-understanding-the-generated-structure) |
| 6 | [Create a Chart Manually](#6-create-a-chart-manually) |
| 7 | [Create `Chart.yaml`](#7-create-chartyaml) |
| 8 | [Understand `Chart.yaml`](#8-understand-chartyaml) |
| 9 | [Create `values.yaml`](#9-create-valuesyaml) |
| 10 | [Understand `values.yaml`](#10-understand-valuesyaml) |
| 11 | [Create `_helpers.tpl`](#11-create-helperstpl) |
| 12 | [Create the Deployment Template](#12-create-the-deployment-template) |
| 13 | [Understand the Deployment Template](#13-understand-the-deployment-template) |
| 14 | [Understanding `.Values`](#14-understanding-values) |
| 15 | [Understanding `.Release`](#15-understanding-release) |
| 16 | [Create the Service Template](#16-create-the-service-template) |
| 17 | [Understand the Service Template](#17-understand-the-service-template) |
| 18 | [Connect the Service to the Deployment](#18-connect-the-service-to-the-deployment) |
| 19 | [Validate the Chart](#19-validate-the-chart) |
| 20 | [Render the Templates](#20-render-the-templates) |
| 21 | [Inspect the Rendered YAML](#21-inspect-the-rendered-yaml) |
| 22 | [Validate with Kubernetes](#22-validate-with-kubernetes) |
| 23 | [Install the Helm Chart](#23-install-the-helm-chart) |
| 24 | [Check the Helm Release](#24-check-the-helm-release) |
| 25 | [Check Kubernetes Resources](#25-check-kubernetes-resources) |
| 26 | [Change Configuration Using `--set`](#26-change-configuration-using---set) |
| 27 | [Use a Custom Values File](#27-use-a-custom-values-file) |
| 28 | [Multiple Values Files](#28-multiple-values-files) |
| 29 | [Environment-Specific Configuration](#29-environment-specific-configuration) |
| 30 | [Upgrade the Application](#30-upgrade-the-application) |
| 31 | [Check Helm History](#31-check-helm-history) |
| 32 | [Roll Back](#32-roll-back) |
| 33 | [Get Helm Values](#33-get-helm-values) |
| 34 | [Get the Installed Manifest](#34-get-the-installed-manifest) |
| 35 | [Uninstall the Release](#35-uninstall-the-release) |
| 36 | [Important Helm Objects](#36-important-helm-objects) |
| 37 | [`.Chart` Object](#37-chart-object) |
| 38 | [`.Template` Object](#38-template-object) |
| 39 | [Helm Template Expressions](#39-helm-template-expressions) |
| 40 | [Important Helm Functions](#40-important-helm-functions) |
| 41 | [`include` and `_helpers.tpl`](#41-include-and-helperstpl) |
| 42 | [Conditional Resources](#42-conditional-resources) |
| 43 | [Looping with `range`](#43-looping-with-range) |
| 44 | [Helm Dependencies](#44-helm-dependencies) |
| 45 | [Recommended Production Structure](#45-recommended-production-structure) |
| 46 | [Common Helm Troubleshooting](#46-common-helm-troubleshooting) |
| 47 | [Helm Workflow](#47-helm-workflow) |
| 48 | [Commands Cheat Sheet](#48-commands-cheat-sheet) |
| 49 | [Final Learning Path](#49-final-learning-path) |
| — | [Conclusion](#conclusion) |

---

# 1. Prerequisites

Before creating a Helm Chart, make sure you have:

```bash
kubectl version --client
helm version
```

You also need access to a Kubernetes cluster.

Check the cluster:

```bash
kubectl get nodes
```

Example:

```text
NAME       STATUS   ROLES           AGE
node-01    Ready    control-plane   10d
node-02    Ready    <none>          10d
```

---

# 2. What Are We Going to Build?

We will create a simple Helm Chart that deploys:

```text
Helm Chart
    │
    ├── Deployment
    │      │
    │      └── NGINX Pod
    │
    └── Service
           │
           └── Exposes NGINX
```

The final chart will look like:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   └── service.yaml
└── .helmignore
```

---

# 3. Create a Project Directory

Create a directory for your Helm projects:

```bash
mkdir helm-projects
cd helm-projects
```

---

# 4. Create a Helm Chart

There are two ways to create a Helm Chart.

## Method 1 — Using `helm create`

The easiest way is:

```bash
helm create myapp
```

This creates a complete starter chart.

Check the directory:

```bash
ls
```

You should see:

```text
myapp
```

Enter the chart:

```bash
cd myapp
```

---

# 5. Understanding the Generated Structure

Run:

```bash
tree
```

You may see:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests/
└── .helmignore
```

For learning, we can remove resources that we do not need.

For example:

```bash
rm templates/hpa.yaml
rm templates/ingress.yaml
rm -rf templates/tests
```

We will keep:

```text
templates/
├── _helpers.tpl
├── deployment.yaml
├── service.yaml
└── serviceaccount.yaml
```

You can also remove `serviceaccount.yaml` if the application does not need a custom ServiceAccount.

---

# 6. Create a Chart Manually

Understanding the manual process is useful.

Instead of:

```bash
helm create myapp
```

we could create the structure ourselves:

```bash
mkdir myapp
cd myapp

mkdir templates
mkdir charts

touch Chart.yaml
touch values.yaml
touch .helmignore
touch templates/deployment.yaml
touch templates/service.yaml
touch templates/_helpers.tpl
```

The structure becomes:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
└── .helmignore
```

For the rest of this guide, we will build the important parts ourselves.

---

# 7. Create Chart.yaml

Create:

```text
Chart.yaml
```

Add:

```yaml
apiVersion: v2

name: myapp

description: A Helm chart for my application

type: application

version: 0.1.0

appVersion: "1.0.0"
```

---

# 8. Understand Chart.yaml

### apiVersion

```yaml
apiVersion: v2
```

This specifies the Helm Chart API version.

For modern Helm 3 application charts, `v2` is normally used.

---

### name

```yaml
name: myapp
```

This is the name of the chart.

---

### description

```yaml
description: A Helm chart for my application
```

A description of the chart.

---

### type

```yaml
type: application
```

This indicates that the chart packages an application.

---

### version

```yaml
version: 0.1.0
```

This is the **Helm Chart version**.

For example:

```text
0.1.0
0.2.0
1.0.0
```

---

### appVersion

```yaml
appVersion: "1.0.0"
```

This represents the application version.

It is separate from the chart version.

For example:

```text
Chart version: 1.2.0
Application version: 3.5.0
```

---

# 9. Create values.yaml

Now create:

```text
values.yaml
```

Add:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 80

resources:
  requests:
    cpu: 100m
    memory: 128Mi

  limits:
    cpu: 500m
    memory: 512Mi
```

---

# 10. Why Do We Need values.yaml?

Without Helm, we might hard-code:

```yaml
replicas: 2

image: nginx:1.27

port: 80
```

With Helm, we move these values into:

```text
values.yaml
```

Then the templates reference them.

For example:

```yaml
replicas: {{ .Values.replicaCount }}
```

This gives us:

```text
values.yaml
      ↓
Helm Template
      ↓
Kubernetes YAML
```

---

# 11. Create _helpers.tpl

Create:

```text
templates/_helpers.tpl
```

Add:

```yaml
{{/*
Create the name of the application.
*/}}
{{- define "myapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}


{{/*
Create the full name of the application.
*/}}
{{- define "myapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}


{{/*
Common labels.
*/}}
{{- define "myapp.labels" -}}
helm.sh/chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}


{{/*
Selector labels.
*/}}
{{- define "myapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "myapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

The `_helpers.tpl` file allows us to reuse common template logic.

---

# 12. Create the Deployment Template

Create:

```text
templates/deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: {{ include "myapp.fullname" . }}

  labels:
    {{- include "myapp.labels" . | nindent 4 }}

spec:

  replicas: {{ .Values.replicaCount }}

  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}

  template:

    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}

    spec:

      containers:

        - name: {{ .Chart.Name }}

          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"

          imagePullPolicy: {{ .Values.image.pullPolicy }}

          ports:

            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP

          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

---

# 13. Understanding the Deployment Template

The important difference between normal Kubernetes YAML and Helm YAML is the use of expressions such as:

```text
{{ .Values.replicaCount }}
```

and:

```text
{{ .Values.image.repository }}
```

For example:

```yaml
replicas: {{ .Values.replicaCount }}
```

comes from:

```yaml
replicaCount: 2
```

in `values.yaml`.

Therefore Helm generates:

```yaml
replicas: 2
```

---

# 14. Understanding `.Values`

`.Values` is the object containing values supplied to Helm.

For example:

```yaml
image:
  repository: nginx
  tag: "1.27"
```

can be accessed using:

```text
.Values.image.repository
```

and:

```text
.Values.image.tag
```

Therefore:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

becomes:

```yaml
image: "nginx:1.27"
```

---

# 15. Understanding `.Release`

Helm also provides release information.

For example:

```text
.Release.Name
```

If we install:

```bash
helm install myapp ./myapp
```

then:

```text
.Release.Name
```

will be:

```text
myapp
```

This allows templates to generate unique resource names.

---

# 16. Create the Service Template

Create:

```text
templates/service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service

metadata:
  name: {{ include "myapp.fullname" . }}

  labels:
    {{- include "myapp.labels" . | nindent 4 }}

spec:

  type: {{ .Values.service.type }}

  ports:

    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
      name: http

  selector:
    {{- include "myapp.selectorLabels" . | nindent 4 }}
```

---

# 17. Understand the Service Template

The Service type comes from:

```yaml
service:
  type: ClusterIP
```

and is referenced with:

```text
.Values.service.type
```

The port comes from:

```yaml
service:
  port: 80
```

and is referenced with:

```text
.Values.service.port
```

This makes the Service configurable.

---

# 18. Final Chart Structure

Our chart now looks like:

```text
myapp/
│
├── Chart.yaml
│
├── values.yaml
│
├── charts/
│
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   └── service.yaml
│
└── .helmignore
```

---

# 19. Validate the Chart

Before installing the chart, run:

```bash
helm lint ./myapp
```

If everything is correct, Helm should report that the chart passed linting.

---

# 20. Render the Templates

Before installing anything, render the templates:

```bash
helm template myapp ./myapp
```

This does not deploy anything.

It simply converts:

```text
Helm Templates
      +
values.yaml
      ↓
Kubernetes YAML
```

You should see:

```yaml
apiVersion: apps/v1
kind: Deployment
...
```

and:

```yaml
apiVersion: v1
kind: Service
...
```

---

# 21. Save the Rendered YAML

You can save the generated YAML:

```bash
helm template myapp ./myapp > rendered.yaml
```

Then inspect:

```bash
cat rendered.yaml
```

This is useful for debugging.

---

# 22. Validate with Kubernetes

You can perform a client-side dry run:

```bash
helm template myapp ./myapp | kubectl apply --dry-run=client -f -
```

This allows you to catch Kubernetes manifest problems before actually deploying.

---

# 23. Install the Helm Chart

Install the chart:

```bash
helm install myapp ./myapp
```

Check the release:

```bash
helm list
```

You should see:

```text
NAME    NAMESPACE   REVISION   STATUS
myapp   default     1          deployed
```

---

# 24. Check the Release

Run:

```bash
helm status myapp
```

This shows information about the Helm release.

---

# 25. Check Kubernetes Resources

Check the Deployment:

```bash
kubectl get deployments
```

Check Pods:

```bash
kubectl get pods
```

Check the Service:

```bash
kubectl get services
```

You should see something similar to:

```text
NAME          READY   UP-TO-DATE   AVAILABLE
myapp-myapp   2/2     2            2
```

---

# 26. Change Configuration Using --set

One of the main benefits of Helm is that we do not need to edit the template.

For example, our default configuration is:

```yaml
replicaCount: 2
```

We can deploy three replicas:

```bash
helm upgrade myapp ./myapp \
  --set replicaCount=3
```

Check:

```bash
kubectl get pods
```

We should now have three Pods.

---

# 27. Change the Image

We can also change the image tag:

```bash
helm upgrade myapp ./myapp \
  --set image.tag=latest
```

Or:

```bash
helm upgrade myapp ./myapp \
  --set image.repository=httpd \
  --set image.tag=2.4
```

The template does not change.

Only the values change.

---

# 28. Using a Separate Values File

Instead of using multiple `--set` arguments, we can create:

```text
values-prod.yaml
```

Example:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.27"

service:
  type: LoadBalancer
  port: 80
  targetPort: 80
```

Install using:

```bash
helm install myapp ./myapp \
  -f values-prod.yaml
```

Or upgrade:

```bash
helm upgrade myapp ./myapp \
  -f values-prod.yaml
```

---

# 29. Environment-Specific Configuration

A common production structure is:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    └── service.yaml
```

Development:

```bash
helm upgrade --install myapp ./myapp \
  -f values-dev.yaml
```

Staging:

```bash
helm upgrade --install myapp ./myapp \
  -f values-staging.yaml
```

Production:

```bash
helm upgrade --install myapp ./myapp \
  -f values-prod.yaml
```

The templates remain the same.

Only configuration changes.

---

# 30. Upgrade the Application

Suppose we change:

```yaml
replicaCount: 3
```

in our values file.

Run:

```bash
helm upgrade myapp ./myapp
```

Check:

```bash
helm status myapp
```

And:

```bash
kubectl get pods
```

---

# 31. Check Helm History

Helm maintains release revisions.

Run:

```bash
helm history myapp
```

Example:

```text
REVISION   STATUS
1          superseded
2          deployed
```

---

# 32. Roll Back

If the latest deployment has a problem:

```bash
helm rollback myapp 1
```

Then check:

```bash
helm history myapp
```

and:

```bash
helm status myapp
```

Rollback is one of the major advantages of managing applications through Helm releases.

---

# 33. View Release Values

To see values used by the release:

```bash
helm get values myapp
```

To see all values:

```bash
helm get values myapp --all
```

---

# 34. View Generated Kubernetes Manifests

Use:

```bash
helm get manifest myapp
```

This shows the Kubernetes resources generated for the installed release.

This is particularly useful when troubleshooting.

---

# 35. Uninstall the Application

To remove the Helm release:

```bash
helm uninstall myapp
```

Check:

```bash
helm list
```

Then:

```bash
kubectl get pods
```

The resources managed by the release should be removed according to Helm/Kubernetes resource behavior.

---

# 36. Complete Workflow

The complete development workflow is:

```text
Create Chart
     │
     ▼
Create Chart.yaml
     │
     ▼
Create values.yaml
     │
     ▼
Create templates
     │
     ▼
helm lint
     │
     ▼
helm template
     │
     ▼
kubectl dry-run
     │
     ▼
helm install
     │
     ▼
Check Kubernetes resources
     │
     ▼
helm upgrade
     │
     ▼
helm history
     │
     ▼
helm rollback
```

---

# 37. Important Helm Concepts

## Chart

A package containing:

```text
Templates
+
Configuration
+
Metadata
```

---

## values.yaml

Contains configurable values:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"
```

---

## templates/

Contains Kubernetes resource templates:

```text
templates/
├── deployment.yaml
└── service.yaml
```

---

## Release

An installed instance of a chart.

For example:

```bash
helm install myapp ./myapp
```

Here:

```text
myapp = Release
./myapp = Chart
```

---

# 38. Important Helm Template Objects

Some of the most commonly used Helm objects are:

### `.Values`

Accesses values from:

```text
values.yaml
```

Example:

```text
.Values.replicaCount
```

---

### `.Release`

Contains release information.

Example:

```text
.Release.Name
```

---

### `.Chart`

Contains chart information.

Example:

```text
.Chart.Name
.Chart.Version
.Chart.AppVersion
```

---

### `.Template`

Contains information about the current template.

---

# 39. Important Helm Functions

Helm provides many template functions.

Some commonly used functions are:

```text
include
default
required
toYaml
quote
nindent
indent
printf
replace
trunc
trimSuffix
```

Example:

```yaml
{{ include "myapp.fullname" . }}
```

Another example:

```yaml
{{ .Values.resources | toYaml | nindent 12 }}
```

---

# 40. Why Use `include`?

Instead of repeating:

```yaml
app.kubernetes.io/name: myapp
app.kubernetes.io/instance: myapp
```

we can define labels in `_helpers.tpl` and reuse them:

```yaml
{{ include "myapp.labels" . }}
```

This makes charts:

* Cleaner
* Reusable
* Easier to maintain

---

# 41. Why Use `nindent`?

Consider:

```yaml
resources:
  {{- toYaml .Values.resources | nindent 12 }}
```

`toYaml` converts an object into YAML.

`nindent` adds indentation and a newline.

This is especially useful when inserting nested YAML structures into Kubernetes templates.

---

# 42. Conditional Resources

Helm can conditionally create resources.

For example:

```yaml
ingress:
  enabled: false
```

Template:

```yaml
{{- if .Values.ingress.enabled }}

apiVersion: networking.k8s.io/v1
kind: Ingress

...

{{- end }}
```

If:

```yaml
ingress:
  enabled: false
```

the Ingress is not generated.

If:

```yaml
ingress:
  enabled: true
```

the Ingress is generated.

---

# 43. Using Required Values

Sometimes a value must be provided.

Helm provides:

```text
required
```

Example:

```yaml
image:
  repository: {{ required "image.repository is required" .Values.image.repository }}
```

If the value is missing, Helm reports an error instead of generating an invalid manifest.

---

# 44. Helm Chart Development Best Practice

A useful workflow is:

```bash
helm lint ./myapp

helm template myapp ./myapp

helm template myapp ./myapp | kubectl apply --dry-run=client -f -

helm upgrade --install myapp ./myapp
```

Then verify:

```bash
kubectl get pods
kubectl get deployment
kubectl get service
```

---

# 45. Recommended Production Structure

A more complete application chart might look like:

```text
myapp/
│
├── Chart.yaml
│
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
│
├── charts/
│
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── ingress.yaml
│   ├── pvc.yaml
│   ├── serviceaccount.yaml
│   └── hpa.yaml
│
└── .helmignore
```

---

# 46. Important Rule

Do not put everything into `values.yaml`.

For example, this is unnecessary:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
```

These are normally part of the template.

Instead, put values that are expected to change:

```yaml
replicaCount: 3

image:
  repository: myapp
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80
```

Think of it as:

```text
templates/
    ↓
How the Kubernetes resource is structured

values.yaml
    ↓
What configuration the application should use
```

---

# 47. Final Example

Our final chart:

```text
myapp/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    └── service.yaml
```

Deployment flow:

```text
                    Helm Chart
                        │
            ┌───────────┴───────────┐
            │                       │
       Chart.yaml              values.yaml
            │                       │
            │                       │
            └───────────┬───────────┘
                        │
                        ▼
                   templates/
                        │
                        ▼
                 Helm Rendering
                        │
                        ▼
              Kubernetes YAML
                        │
                        ▼
                Kubernetes API
                        │
                        ▼
              Deployment + Pods
                        │
                        ▼
                     Service
```

---

# 48. Commands Cheat Sheet

## Create

```bash
helm create myapp
```

## Validate

```bash
helm lint ./myapp
```

## Render

```bash
helm template myapp ./myapp
```

## Install

```bash
helm install myapp ./myapp
```

## Install with values

```bash
helm install myapp ./myapp -f values-prod.yaml
```

## Upgrade

```bash
helm upgrade myapp ./myapp
```

## Upgrade or install

```bash
helm upgrade --install myapp ./myapp
```

## List releases

```bash
helm list
```

## Status

```bash
helm status myapp
```

## History

```bash
helm history myapp
```

## Rollback

```bash
helm rollback myapp 1
```

## Get values

```bash
helm get values myapp
```

## Get manifest

```bash
helm get manifest myapp
```

## Uninstall

```bash
helm uninstall myapp
```

---

# 49. Final Learning Path

To properly learn Helm, follow this sequence:

```text
1. Understand Kubernetes YAML
             ↓
2. Understand Helm
             ↓
3. Create a Chart
             ↓
4. Understand Chart.yaml
             ↓
5. Understand values.yaml
             ↓
6. Learn Helm templates
             ↓
7. Learn .Values
             ↓
8. Learn .Release
             ↓
9. Learn _helpers.tpl
             ↓
10. Create Deployment template
             ↓
11. Create Service template
             ↓
12. Learn ConfigMaps
             ↓
13. Learn Secrets
             ↓
14. Learn Ingress
             ↓
15. Learn conditional templates
             ↓
16. Learn environment-specific values
             ↓
17. Learn helm lint
             ↓
18. Learn helm template
             ↓
19. Learn helm install
             ↓
20. Learn helm upgrade
             ↓
21. Learn helm rollback
             ↓
22. Learn dependencies
             ↓
23. Learn Helm hooks
             ↓
24. Build production-ready charts
```

---

# Conclusion

Creating a Helm Chart from scratch is essentially about separating **Kubernetes resource structure** from **application configuration**.

The core idea is:

```text
Kubernetes Resource
        │
        ▼
     Template
        │
        +
     values.yaml
        │
        ▼
   Helm Rendering
        │
        ▼
Kubernetes Manifest
        │
        ▼
 Kubernetes Cluster
```

The most important concepts to remember are:

```text
Chart.yaml
    → Chart metadata

values.yaml
    → Configuration

templates/
    → Kubernetes resource templates

_helpers.tpl
    → Reusable template functions

helm install
    → Create a release

helm upgrade
    → Update a release

helm rollback
    → Return to an earlier revision
```

Once you understand these concepts, you can take almost any Kubernetes application and package it into a reusable Helm Chart.
