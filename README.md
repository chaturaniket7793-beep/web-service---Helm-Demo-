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
Step 6: Create GitHub Repository

Create a new GitHub repository with the following name:

WebService-Repo

Do not initialize the repository with a README, .gitignore, or license if you are going to push the existing local project.

Initialize Git

From the web-service directory:

git init
Add GitHub Remote

Use your GitHub username:

git remote add origin https://github.com/chaturaniket7793-beep/WebService-Repo.git
Add Files
git add .
Commit
git commit -m "Add webservice Helm chart"
Push to GitHub
git branch -M main
git push -u origin main

Your GitHub repository will be:

https://github.com/chaturaniket7793-beep/WebService-Repo
Step 7: Configure GitHub Pages

Open the WebService-Repo repository on GitHub.

Go to:

Settings → Pages

Under Build and deployment:

Source: Deploy from a branch
Branch: main
Folder: / (root)

Click Save.

Your Helm repository URL will be:

https://chaturaniket7793-beep.github.io/WebService-Repo/

Important: The repository root must contain the Helm repository files:

index.yaml
webservice-0.1.0.tgz

These files are created using:

helm package webservice
helm repo index .
Step 8: Add the Helm Repository

Add the GitHub Pages Helm repository:

helm repo add myrepo https://chaturaniket7793-beep.github.io/WebService-Repo/

Update the repository:

helm repo update

Verify the repository:

helm repo list

Expected output should contain:

myrepo    https://chaturaniket7793-beep.github.io/WebService-Repo/
Step 9: Search the Helm Repository

Run:

helm search repo myrepo

Expected output:

NAME              CHART VERSION    APP VERSION    DESCRIPTION
myrepo/webservice 0.1.0             1.0            A Helm chart for Kubernetes

The important part is:

myrepo/webservice
Step 10: Install the Helm Chart

Install the Helm chart:

helm install webservice-service myrepo/webservice

Expected result:

NAME: webservice-service
STATUS: deployed
Step 11: Verify the Deployment
Check Helm Release
helm list

You should see:

NAME                  STATUS
webservice-service    deployed
Check Pods
kubectl get pods

Expected:

NAME                            READY   STATUS    RESTARTS   AGE
webservice-xxxxxxxxxx-xxxxx    1/1     Running   0          ...
Check Deployment
kubectl get deployment

Expected:

NAME         READY   UP-TO-DATE   AVAILABLE
webservice   1/1     1            1
Check Service
kubectl get svc

Expected:

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
webservice   ClusterIP   10.x.x.x        <none>        80/TCP
Step 12: Test the Application

The Service is configured as:

ClusterIP

Therefore, it is accessible inside the Kubernetes cluster.

Use port forwarding to access it from your local machine:

kubectl port-forward svc/webservice 8080:80

Expected output:

Forwarding from 127.0.0.1:8080 -> 80

Open your browser and visit:

http://localhost:8080

You should see the Nginx welcome page.

Complete Workflow
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
helm repo index .
       ↓
Create GitHub Repository
       ↓
git init
       ↓
git add .
       ↓
git commit
       ↓
git push
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
kubectl get deployment
       ↓
kubectl get svc
       ↓
kubectl port-forward
       ↓
Open Nginx Application
Important Commands
Helm Chart
helm create webservice

helm lint webservice

helm template webservice ./webservice

helm package webservice

helm repo index .
GitHub
git init

git remote add origin https://github.com/chaturaniket7793-beep/WebService-Repo.git

git add .

git commit -m "Add webservice Helm chart"

git branch -M main

git push -u origin main
Helm Repository
helm repo add myrepo https://chaturaniket7793-beep.github.io/WebService-Repo/

helm repo update

helm search repo myrepo
Install
helm install webservice-service myrepo/webservice
Kubernetes Verification
kubectl get pods

kubectl get deployment

kubectl get svc
Application Testing
kubectl port-forward svc/webservice 8080:80

Open:

http://localhost:8080
Final Repository Structure

Your GitHub repository should look approximately like this:

WebService-Repo/
│
├── webservice/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── .helmignore
│   │
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       ├── serviceaccount.yaml
│       ├── _helpers.tpl
│       └── tests/
│           └── test-connection.yaml
│
├── webservice-0.1.0.tgz
│
├── index.yaml
│
└── README.md

Important: index.yaml and webservice-0.1.0.tgz should be in the root of the GitHub repository, because GitHub Pages is serving the root as the Helm repository.
