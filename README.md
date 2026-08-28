# Web-Service — Helm Demo

## Creating a Helm Repository for Web Services

### Scenario

As a **DevOps Engineer**, you manage web services and want to streamline their **deployment and scaling**.

In this demo, we will:

* Create a Helm chart for a web service.
* Use the **Nginx** image.
* Customize the Deployment and Service templates.
* Package the Helm chart.
* Host the Helm repository on **GitHub Pages**.
* Install the chart using the Helm repository.

---

# Helm Prerequisites

* Helm installed
* Kubernetes cluster running
* `kubectl` installed
* `kubectl` configured
* Git installed
* GitHub account
* Basic Kubernetes knowledge
* Basic Git/GitHub knowledge

---

# Step 1: Create the Helm Chart

### Create the project directory

```bash
mkdir web-service
cd web-service
```

### Create the Helm chart

```bash
helm create webservice
```

---

# Step 2: Review Helm Chart Structure

```text
webservice/
├── .helmignore
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
    ├── _helpers.tpl
    ├── NOTES.txt
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── serviceaccount.yaml
    ├── hpa.yaml
    └── tests/
        └── test-connection.yaml
```

### Important Files

* `Chart.yaml` → Chart information
* `values.yaml` → Default configuration values
* `templates/deployment.yaml` → Kubernetes Deployment
* `templates/service.yaml` → Kubernetes Service
* `templates/ingress.yaml` → Ingress configuration
* `templates/serviceaccount.yaml` → ServiceAccount
* `templates/hpa.yaml` → Horizontal Pod Autoscaler
* `templates/_helpers.tpl` → Reusable Helm templates

---

# Step 3: Customize the Helm Chart

## 3.1 Deployment Template

File:

```text
webservice/templates/deployment.yaml
```

### Requirements

* Set the replica count.
* Use values from `values.yaml`.
* Use `{{- with .Values.image }}` for image configuration.
* Define the replica count variable.
* Add required labels.
* Use the same labels for the Deployment selector and Pod selector.
* Ensure the Deployment works with the Nginx Service.

### `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
  labels:
    app.kubernetes.io/name: {{ .Chart.Name }}
spec:
  {{- $replicaCount := .Values.replicaCount }}
  replicas: {{ $replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ .Chart.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          {{- with .Values.image }}
          image: "{{ .repository }}:{{ .tag }}"
          imagePullPolicy: IfNotPresent
          {{- end }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

---

## 3.2 Service Template

File:

```text
webservice/templates/service.yaml
```

### Requirements

* Create the `service.yaml` file.
* Use `ClusterIP` service.
* Expose port `80`.
* Target Nginx container port `80`.
* Use TCP protocol.
* Use the same label as the Deployment.

### `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
  labels:
    app.kubernetes.io/name: {{ .Chart.Name }}
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: {{ .Chart.Name }}
```

---

## 3.3 Values File

File:

```text
webservice/values.yaml
```

### `values.yaml`

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: latest
```

---

# Step 4: Validate the Helm Chart

### Go to the project directory

```bash
cd web-service
```

### Check the chart

```bash
helm lint webservice
```

### Render the Kubernetes manifests

```bash
helm template webservice ./webservice
```

### Optional: Check the generated resources

```bash
helm template webservice ./webservice | kubectl apply --dry-run=client -f -
```

---

# Step 5: Package the Helm Chart

### Make sure you are in the parent directory

```text
web-service/
└── webservice/
```

Run:

```bash
helm package webservice
```

This creates a package similar to:

```text
webservice-0.1.0.tgz
```

### Create the Helm repository index

Run from the **same parent directory**:

```bash
helm repo index .
```

You should now have:

```text
web-service/
├── webservice/
├── webservice-0.1.0.tgz
└── index.yaml
```

---

# Step 6: Create GitHub Repository

Create a new GitHub repository.

### Repository Name

```text
WebService-Repo
```

### Initialize Git

From the `web-service` directory:

```bash
git init
```

### Add GitHub remote

Replace `username` with your GitHub username:

```bash
git remote add origin https://github.com/username/WebService-Repo.git
```

### Add files

```bash
git add .
```

### Commit

```bash
git commit -m "Add webservice Helm chart"
```

### Push to GitHub

```bash
git branch -M main
git push -u origin main
```

---

# Step 7: Configure GitHub Pages

* Open **WebService-Repo** on GitHub.
* Go to **Settings**.
* Select **Pages**.
* Under **Build and deployment**, select:

  * Source: **Deploy from a branch**
  * Branch: **main**
  * Folder: **/ (root)**
* Click **Save**.

Your Helm repository URL will be:

```text
https://username.github.io/WebService-Repo/
```

---

# Step 8: Add the Helm Repository

Run:

```bash
helm repo add myrepo https://username.github.io/WebService-Repo/
```

Update the repository:

```bash
helm repo update
```

---

# Step 9: Search the Helm Repository

```bash
helm search repo myrepo
```

Expected chart:

```text
myrepo/webservice
```

---

# Step 10: Install the Helm Chart

Install the chart:

```bash
helm install webservice-service myrepo/webservice
```

---

# Step 11: Verify the Deployment

### Check Helm release

```bash
helm list
```

### Check Pods

```bash
kubectl get pods
```

### Check Deployment

```bash
kubectl get deployment
```

### Check Service

```bash
kubectl get svc
```

---

# Step 12: Test the Application

Because the Service type is `ClusterIP`, access it using port forwarding:

```bash
kubectl port-forward svc/webservice 8080:80
```

Open:

```text
http://localhost:8080
```

You should see the **Nginx welcome page**.

---

# Complete Workflow

```text
Create Helm Chart
       ↓
Customize Deployment
       ↓
Create Service
       ↓
Update values.yaml
       ↓
helm lint
       ↓
helm template
       ↓
helm package
       ↓
helm repo index
       ↓
Push to GitHub
       ↓
Enable GitHub Pages
       ↓
helm repo add
       ↓
helm repo update
       ↓
helm search repo
       ↓
helm install
       ↓
kubectl get pods
       ↓
kubectl get svc
```

# Important Commands

```bash
helm create webservice

helm lint webservice

helm template webservice ./webservice

helm package webservice

helm repo index .

git init

git add .

git commit -m "Add webservice Helm chart"

git branch -M main

git push -u origin main

helm repo add myrepo https://username.github.io/WebService-Repo/

helm repo update

helm search repo myrepo

helm install webservice-service myrepo/webservice

kubectl get pods

kubectl get svc
```
