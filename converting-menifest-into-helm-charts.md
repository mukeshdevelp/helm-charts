# Converting Existing Kubernetes Manifests into a Helm Chart

This guide explains how to convert an existing set of Kubernetes manifest files into a reusable **Helm Chart**.

The starting point is an application that is already deployed or can be deployed using normal Kubernetes YAML files.

---

# 1. Starting Point

Suppose we currently have Kubernetes manifests like:

```text
kubernetes/
├── namespace.yaml
├── deployment.yaml
├── service.yaml
├── configmap.yaml
├── secret.yaml
├── ingress.yaml
├── pvc.yaml
└── serviceaccount.yaml
```

For example, the `deployment.yaml` may contain:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp

spec:
  replicas: 3

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp

    spec:
      containers:
        - name: myapp
          image: nginx:1.27
          ports:
            - containerPort: 80
```

The goal is to convert this into:

```text
myapp-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── ingress.yaml
│   ├── pvc.yaml
│   └── serviceaccount.yaml
└── .helmignore
```

---

# 2. Understand What Helm Actually Does

It is important to understand that Helm does **not** convert Kubernetes YAML into some completely different Kubernetes format.

Helm still produces normal Kubernetes YAML.

The difference is that Helm introduces:

```text
Templates
   +
Values
   ↓
Rendered Kubernetes YAML
   ↓
Kubernetes API
```

For example:

```text
values.yaml
     ↓
Helm template
     ↓
deployment.yaml
     ↓
Rendered Kubernetes YAML
     ↓
Kubernetes
```

So the Kubernetes resources themselves do not disappear.

They become **templates**.

---

# 3. Step 1 — Inventory the Existing Manifests

Before creating the Helm chart, identify all the Kubernetes resources.

For example:

```text
namespace.yaml
deployment.yaml
service.yaml
configmap.yaml
secret.yaml
ingress.yaml
pvc.yaml
serviceaccount.yaml
```

Create a list:

```text
Namespace
Deployment
Service
ConfigMap
Secret
Ingress
PVC
ServiceAccount
```

This helps ensure that no resource is accidentally missed during the conversion.

---

# 4. Step 2 — Group Resources by Application

If the Kubernetes cluster contains multiple applications, do not blindly put every manifest into one Helm chart.

For example:

```text
Kubernetes Cluster
│
├── frontend
│   ├── Deployment
│   ├── Service
│   └── Ingress
│
├── backend
│   ├── Deployment
│   ├── Service
│   └── ConfigMap
│
└── worker
    ├── Deployment
    └── ConfigMap
```

You might create:

```text
frontend-chart/
backend-chart/
worker-chart/
```

Alternatively, if they are tightly coupled parts of one application, they can be managed by one chart.

The chart boundary should generally follow the **application lifecycle**.

---

# 5. Step 3 — Create a Helm Chart

Use:

```bash
helm create myapp
```

This creates:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   ├── serviceaccount.yaml
│   ├── tests/
│   └── _helpers.tpl
└── .helmignore
```

The generated chart contains example resources.

If you are converting an existing application, you can remove the example templates that you do not need.

For example:

```text
templates/
├── deployment.yaml
├── service.yaml
├── ingress.yaml
└── _helpers.tpl
```

---

# 6. Step 4 — Understand Chart.yaml

Open:

```text
Chart.yaml
```

Example:

```yaml
apiVersion: v2

name: myapp

description: Helm chart for my application

type: application

version: 0.1.0

appVersion: "1.0.0"
```

Important fields:

```text
name
    ↓
Name of the Helm chart

version
    ↓
Version of the Helm chart

appVersion
    ↓
Version of the application
```

---

# 7. Step 5 — Move Existing Manifests into templates/

Take the existing Kubernetes manifests and move the relevant files into:

```text
templates/
```

For example:

```text
Before:

kubernetes/
├── deployment.yaml
├── service.yaml
├── configmap.yaml
└── ingress.yaml
```

After:

```text
myapp/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    └── ingress.yaml
```

At this point, the files can still contain the original hard-coded values.

The next step is to convert those hard-coded values into Helm values.

---

# 8. Step 6 — Identify Hard-Coded Values

This is one of the most important steps.

Look through the existing manifests and identify values that may change between environments or deployments.

For example:

```yaml
replicas: 3
```

```yaml
image: nginx:1.27
```

```yaml
containerPort: 8080
```

```yaml
service:
  type: LoadBalancer
```

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

These are candidates for `values.yaml`.

---

# 9. Step 7 — Create values.yaml

Move configurable values into:

```text
values.yaml
```

For example:

```yaml
replicaCount: 3

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
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

The purpose is to separate:

```text
Application configuration
        from
Kubernetes templates
```

---

# 10. Step 8 — Convert the Deployment into a Template

Original Kubernetes manifest:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp

spec:
  replicas: 3

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp

    spec:
      containers:
        - name: myapp

          image: nginx:1.27

          ports:
            - containerPort: 80
```

We identify:

```text
3
nginx
1.27
80
```

as values that may be configurable.

The Helm version becomes:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: {{ include "myapp.fullname" . }}

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
            - containerPort: {{ .Values.service.targetPort }}
```

Now the Kubernetes resource is reusable.

---

# 11. Step 9 — Convert the Service

Original:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: myapp

spec:
  selector:
    app: myapp

  ports:
    - port: 80
      targetPort: 80

  type: ClusterIP
```

Helm version:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: {{ include "myapp.fullname" . }}

spec:
  type: {{ .Values.service.type }}

  selector:
    {{- include "myapp.selectorLabels" . | nindent 4 }}

  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
```

The values now come from:

```yaml
service:
  type: ClusterIP
  port: 80
  targetPort: 80
```

---

# 12. Step 10 — Convert ConfigMaps

Suppose the original ConfigMap is:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: myapp-config

data:
  LOG_LEVEL: "info"
  API_URL: "http://backend:8080"
```

We can move the configuration into:

```yaml
config:
  LOG_LEVEL: "info"
  API_URL: "http://backend:8080"
```

Then create:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: {{ include "myapp.fullname" . }}-config

data:
  {{- toYaml .Values.config | nindent 2 }}
```

Now the configuration can be changed without modifying the template.

---

# 13. Step 11 — Handle Secrets Carefully

Suppose the original manifest contains:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: myapp-secret

data:
  DB_USERNAME: bXl1c2Vy
  DB_PASSWORD: cGFzc3dvcmQ=
```

Do **not** simply move production passwords into a Git-tracked `values.yaml`.

Instead, consider using:

* Kubernetes Secrets
* External Secrets
* AWS Secrets Manager
* HashiCorp Vault
* Sealed Secrets
* Another appropriate secret-management solution

For example, the chart could reference existing secrets:

```yaml
existingSecret: myapp-secret
```

and the Deployment can use:

```yaml
envFrom:
  - secretRef:
      name: {{ .Values.existingSecret }}
```

This keeps sensitive credentials outside the Helm chart.

---

# 14. Step 12 — Convert Ingress

Original:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: myapp

spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp
                port:
                  number: 80
```

Move the host into:

```yaml
ingress:
  enabled: true

  host: myapp.example.com

  path: /

  pathType: Prefix
```

Then template it:

```yaml
{{- if .Values.ingress.enabled }}

apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: {{ include "myapp.fullname" . }}

spec:
  rules:
    - host: {{ .Values.ingress.host }}

      http:
        paths:
          - path: {{ .Values.ingress.path }}

            pathType: {{ .Values.ingress.pathType }}

            backend:
              service:
                name: {{ include "myapp.fullname" . }}

                port:
                  number: {{ .Values.service.port }}

{{- end }}
```

The important part is:

```text
{{- if .Values.ingress.enabled }}
```

This allows the Ingress to be enabled or disabled through configuration.

---

# 15. Step 13 — Handle PersistentVolumes and PVCs

Suppose the existing application has:

```text
PersistentVolume
PersistentVolumeClaim
```

Determine whether the PV should actually be managed by the chart.

In many applications, you may only want the Helm chart to manage the PVC while the underlying storage is managed by the cluster/cloud provider.

For example:

```yaml
persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 10Gi
  storageClass: gp3
```

Then the PVC template can use:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: {{ include "myapp.fullname" . }}

spec:
  accessModes:
    - {{ .Values.persistence.accessMode }}

  resources:
    requests:
      storage: {{ .Values.persistence.size }}

  storageClassName: {{ .Values.persistence.storageClass }}
```

The exact approach depends on how storage is provisioned in your cluster.

---

# 16. Step 14 — Use _helpers.tpl

Helm charts commonly use:

```text
templates/_helpers.tpl
```

to avoid repeating names and labels.

For example:

```yaml
{{- define "myapp.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end }}
```

Then:

```yaml
name: {{ include "myapp.fullname" . }}
```

can be used throughout the chart.

Similarly, common labels can be defined:

```yaml
{{- define "myapp.labels" -}}
app.kubernetes.io/name: {{ include "myapp.fullname" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```

This keeps the chart consistent.

---

# 17. Step 15 — Compare the Original and Helm Version

At this stage you should have:

```text
Original Kubernetes manifests
            ↓
Converted Helm templates
            ↓
values.yaml
```

For example:

```text
Original:

replicas: 3
image: nginx:1.27
```

Helm:

```yaml
replicas: {{ .Values.replicaCount }}

image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

Values:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.27"
```

---

# 18. Step 16 — Render the Chart

Before installing anything, render the templates:

```bash
helm template myapp ./myapp
```

This generates the final Kubernetes YAML.

You should verify that:

```text
Helm template
      ↓
Produces valid Kubernetes YAML
```

You can save the output:

```bash
helm template myapp ./myapp > rendered.yaml
```

Then inspect:

```bash
cat rendered.yaml
```

---

# 19. Step 17 — Lint the Chart

Run:

```bash
helm lint ./myapp
```

A successful result should indicate that the chart passed Helm's lint checks.

This should be done before installation.

---

# 20. Step 18 — Validate Against Kubernetes

You can also use Kubernetes validation:

```bash
helm template myapp ./myapp | kubectl apply --dry-run=client -f -
```

This allows you to check the generated manifests without actually applying them.

The workflow becomes:

```text
helm lint
    ↓
helm template
    ↓
kubectl dry-run
    ↓
helm install
```

---

# 21. Step 19 — Install the Helm Chart

Once the chart is validated:

```bash
helm install myapp ./myapp
```

Check the release:

```bash
helm list
```

Check its status:

```bash
helm status myapp
```

Then check Kubernetes:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

---

# 22. Step 20 — Compare With the Original Deployment

If the application was already running using raw YAML, compare the old and new deployments.

Check:

```bash
kubectl get deployment
kubectl get service
kubectl get configmap
kubectl get secret
kubectl get ingress
kubectl get pvc
```

Make sure the Helm deployment has the expected resources.

---

# 23. Step 21 — Create Environment-Specific Values

Once the basic chart works, create environment-specific files.

For example:

```text
myapp/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
└── templates/
```

### values-dev.yaml

```yaml
replicaCount: 1

image:
  repository: myapp
  tag: dev

ingress:
  enabled: false
```

### values-prod.yaml

```yaml
replicaCount: 3

image:
  repository: myapp
  tag: "1.0.0"

ingress:
  enabled: true
  host: myapp.example.com
```

Deploy development:

```bash
helm upgrade --install myapp ./myapp \
  -f values-dev.yaml
```

Deploy production:

```bash
helm upgrade --install myapp ./myapp \
  -f values-prod.yaml
```

The templates remain the same.

Only the configuration changes.

---

# 24. Final Helm Chart Structure

After conversion, the project could look like:

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
│   └── serviceaccount.yaml
│
└── .helmignore
```

---

# 25. Complete Conversion Flow

The complete process is:

```text
Existing Kubernetes Manifests
              │
              ▼
      Inventory Resources
              │
              ▼
      Group by Application
              │
              ▼
        helm create myapp
              │
              ▼
       Move manifests into
          templates/
              │
              ▼
    Identify hard-coded values
              │
              ▼
         Create values.yaml
              │
              ▼
    Replace hard-coded values
       with .Values references
              │
              ▼
        Create _helpers.tpl
              │
              ▼
       Handle Secrets safely
              │
              ▼
        helm lint ./myapp
              │
              ▼
   helm template myapp ./myapp
              │
              ▼
      Kubernetes dry-run
              │
              ▼
      helm install / upgrade
              │
              ▼
      Verify Kubernetes resources
              │
              ▼
    Create environment-specific
          values files
```

---

# 26. Example Before and After

## Before

A normal Kubernetes deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend

spec:
  replicas: 3

  template:
    spec:
      containers:
        - name: backend
          image: mycompany/backend:1.0.0
          ports:
            - containerPort: 8000
```

Everything is hard-coded.

---

## After

### values.yaml

```yaml
replicaCount: 3

image:
  repository: mycompany/backend
  tag: "1.0.0"

containerPort: 8000
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: {{ include "myapp.fullname" . }}

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

        - name: backend

          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"

          ports:
            - containerPort: {{ .Values.containerPort }}
```

Now we can deploy:

```bash
helm install backend ./myapp
```

or change the image without modifying the template:

```bash
helm upgrade backend ./myapp \
  --set image.tag=1.1.0
```

---

# 27. Important Principle

Do **not** turn every single line of a Kubernetes manifest into a Helm variable.

For example, this does not make sense:

```yaml
apiVersion: {{ .Values.apiVersion }}
kind: {{ .Values.kind }}
```

These values normally should remain fixed.

Instead, parameterize values that are expected to change.

Good candidates include:

```text
Image repository
Image tag
Replica count
Container ports
Service type
Ingress hostname
Resource requests
Resource limits
Storage size
Storage class
Environment variables
Feature flags
Application configuration
```

Keep stable Kubernetes structure inside the template.

---

# 28. What Should Usually Go Into values.yaml?

Good candidates:

```yaml
replicaCount: 3

image:
  repository: myapp
  tag: "1.0.0"

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    cpu: 100m
    memory: 128Mi

  limits:
    cpu: 500m
    memory: 512Mi

ingress:
  enabled: true
  host: myapp.example.com
```

Avoid putting sensitive production credentials directly into:

```text
values.yaml
```

especially if the chart is stored in Git.

---

# 29. Helm Conversion Checklist

Use this checklist when converting an existing application.

```text
[ ] Identify all Kubernetes manifests

[ ] Identify which manifests belong to the application

[ ] Create Helm chart

[ ] Create Chart.yaml

[ ] Move Kubernetes resources into templates/

[ ] Identify hard-coded configuration

[ ] Create values.yaml

[ ] Replace configurable values with .Values

[ ] Create reusable helpers in _helpers.tpl

[ ] Add conditional resources where appropriate

[ ] Handle Secrets securely

[ ] Check labels and selectors

[ ] Check resource names

[ ] Check namespaces

[ ] Check ConfigMaps

[ ] Check PVCs

[ ] Check Ingress

[ ] Run helm lint

[ ] Run helm template

[ ] Review rendered YAML

[ ] Run Kubernetes dry-run

[ ] Install the chart

[ ] Verify Pods

[ ] Verify Services

[ ] Verify Ingress

[ ] Verify ConfigMaps/Secrets

[ ] Verify PVCs

[ ] Test helm upgrade

[ ] Test helm rollback

[ ] Create environment-specific values files
```

---

# 30. Important Helm Commands

### Create chart

```bash
helm create myapp
```

### Validate chart

```bash
helm lint ./myapp
```

### Render templates

```bash
helm template myapp ./myapp
```

### Render with a values file

```bash
helm template myapp ./myapp \
  -f values-prod.yaml
```

### Install

```bash
helm install myapp ./myapp
```

### Install with values

```bash
helm install myapp ./myapp \
  -f values-prod.yaml
```

### Upgrade

```bash
helm upgrade myapp ./myapp
```

### Upgrade or install

```bash
helm upgrade --install myapp ./myapp
```

### Check release

```bash
helm status myapp
```

### View values

```bash
helm get values myapp
```

### View generated manifest

```bash
helm get manifest myapp
```

### View release history

```bash
helm history myapp
```

### Rollback

```bash
helm rollback myapp <revision>
```

### Uninstall

```bash
helm uninstall myapp
```

---

# 31. Final Architecture

The important difference is:

## Before Helm

```text
Kubernetes YAML
      │
      ├── Deployment
      ├── Service
      ├── ConfigMap
      ├── Secret
      └── Ingress
              │
              ▼
        Kubernetes API
```

## After Helm

```text
                Helm Chart
                    │
        ┌───────────┴───────────┐
        │                       │
    templates/              values.yaml
        │                       │
        └───────────┬───────────┘
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
        Kubernetes Resources
```

---

# Conclusion

Converting existing Kubernetes manifests into a Helm chart is **not about rewriting Kubernetes resources from scratch**.

The main process is:

```text
Existing YAML
     ↓
Organize resources
     ↓
Create Helm chart
     ↓
Move manifests into templates/
     ↓
Identify configurable values
     ↓
Move them into values.yaml
     ↓
Replace hard-coded values with Helm expressions
     ↓
Add helpers and conditions
     ↓
Validate rendered YAML
     ↓
Install with Helm
```

The key concept to remember is:

> **`templates/` contains the Kubernetes resource structure, while `values.yaml` contains the configuration that changes between deployments.**

Once this separation is understood, converting an existing Kubernetes application into a reusable Helm chart becomes much easier.
