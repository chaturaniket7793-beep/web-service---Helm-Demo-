# WebService — Helm Demo

## Overview

This project demonstrates how to create, package, host, and install a **Helm chart for an Nginx web service** using Kubernetes and GitHub Pages.

### Workflow

```text
Helm Chart → Customize → Validate → Package
→ GitHub → GitHub Pages → Helm Repository
→ Install → Verify → Access Nginx
```

---

## Prerequisites

* Helm
* Kubernetes cluster
* kubectl
* Git
* GitHub account

Check:

```bash
helm version
kubectl get nodes
git --version
```

---

## Step 1: Create Helm Chart

```bash
mkdir web-service
cd web-service
helm create webservice
```

Structure:

```text
web-service/
└── webservice/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── deployment.yaml
        └── service.yaml
```

---

## Step 2: Configure `values.yaml`

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: latest
```

---

## Step 3: Configure Deployment

`webservice/templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

---

## Step 4: Configure Service

`webservice/templates/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
spec:
  type: ClusterIP
  selector:
    app: {{ .Chart.Name }}
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

---

## Step 5: Validate and Package

From the `web-service` directory:

```bash
helm lint webservice
helm template webservice ./webservice
helm package webservice
helm repo index .
```

This creates:

```text
webservice-0.1.0.tgz
index.yaml
```

---

## Step 6: Push to GitHub

Create a GitHub repository:

```text
WebService-Repo
```

Then:

```bash
git init
git remote add origin https://github.com/chaturaniket7793-beep/WebService-Repo.git
git add .
git commit -m "Add webservice Helm chart"
git branch -M main
git push -u origin main
```

---

## Step 7: Enable GitHub Pages

Go to:

```text
GitHub Repository
→ Settings
→ Pages
→ Deploy from a branch
→ main
→ / (root)
→ Save
```

Repository URL:

```text
https://github.com/chaturaniket7793-beep/WebService-Repo
```

Helm repository URL:

```text
https://chaturaniket7793-beep.github.io/WebService-Repo/
```

---

## Step 8: Add Helm Repository

```bash
helm repo add myrepo https://chaturaniket7793-beep.github.io/WebService-Repo/
helm repo update
helm search repo myrepo
```

Expected chart:

```text
myrepo/webservice
```

---

## Step 9: Install Chart

```bash
helm install webservice-service myrepo/webservice
```

Verify:

```bash
helm list
kubectl get pods
kubectl get deployment
kubectl get svc
```

---

## Step 10: Access Nginx

Because the Service is `ClusterIP`, use port forwarding:

```bash
kubectl port-forward svc/webservice 8080:80
```

Open:

```text
http://localhost:8080
```

You should see the **Nginx welcome page**.

---

## Final Repository Structure

```text
WebService-Repo/
├── webservice/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       └── service.yaml
├── webservice-0.1.0.tgz
├── index.yaml
└── README.md
```

## Cleanup

```bash
helm uninstall webservice-service
```
