 web-service- Helm-Demo
___________________________________________________________________________________________________________________________________________________________________
Creating Helm Repositories for Web Services
Scenario
As a DevOps engineer , you manage web services: To streamline deployment and Scaling , we will create a Helm repository containing Helm charts for web services. We will use a nginx image for web services to simulate their functionality.

*******************************************************************************************************************************************************************
Helm Prerequisites
______________________________________
Helm installed
Kubernetes cluster running
kubectl installed
kubectl configured
GitHub account
Git installed
Basic Kubernetes knowledge
Basic Git/GitHub knowledge

*******************************************************************************************************************************************************************

Step 1: Create the Helm Charts
Create the directory structure:
# mkdir web-service
# cd web-service
Generate Helm charts for web services:
command :- # helm create <chartname>
# helm create webservice

******************************************************************************************************************************************************************
Step 2 : Review Helm Charts components

webservice/                 # Root chart directory created by `helm create`
├── .helmignore             # Patterns to ignore when packaging
├── Chart.yaml              # Chart metadata (name, version, description)
├── values.yaml             # Default configuration values
├── charts/                 # Folder for storing chart dependencies (subcharts)
└── templates/              # Folder for Kubernetes manifest templates
    ├── _helpers.tpl        # Reusable template snippets/helpers
    ├── NOTES.txt           # Usage instructions displayed after installation
    ├── deployment.yaml     # Default Kubernetes Deployment manifest
    ├── service.yaml        # Default Kubernetes Service manifest
    ├── ingress.yaml        # Default Kubernetes Ingress manifest
    ├── serviceaccount.yaml # Default Kubernetes ServiceAccount manifest
    ├── hpa.yaml            # Default Horizontal Pod Autoscaler manifest
    └── tests/              # Folder containing test connection scripts
        └── test-connection.yaml

*******************************************************************************************************************************************************************

Step 3: Customize the Helm Charts

nginx Deployment Template (templates/deployment.yaml)
Update the deployment.yaml file.
_________________________________________________________________
Set the required replica count.
Use {{- with .Values.image }} for image-related values.
Define variables where required.
Add all mandatory labels.
Use values from values.yaml.
Make sure the YAML indentation and Helm syntax are correct.
Ensure the Deployment works correctly with the nginx service.


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

____________________________________________________________________________________________________________________________________________________________

Service Template (templates/service.yaml)

Create the missing service.yaml file inside your templates/ directory to expose your Nginx pods within the cluster:

YAML
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

______________________________________________________________________________________________________________________________________________________________

Values File (values.yaml)
Provide a default values.yaml file structured as follows:

replicaCount: 3
image:
  repository: nginx
  tag: latest
*******************************************************************************************************************************************************************

Step 3: Package the Charts
Package the webservice chart: # cd - (come back to main webservice directory and run , Do not run below command inside directory chart files)
# helm package webservice

Create an index file
# helm repo index .


Step 4: Host the Helm Repo on GitHub
Create a new GitHub repository:
Name: WebService-Repo

Initialize a Git repo:
# git init
# git remote add origin https://github.com/username/WebService-repo.git

Push the Helm charts to GitHub:
# git add .
# git commit -m "Add webservice charts"
# git push -u origin main

Configure GitHub Pages:
Go to the repo settings.
Enable GitHub Pages from the main branch.

Step 5: Using the Helm Repo

Add the Helm repo:
# helm repo add myrepo https://username.github.io/helm-repo
# helm repo update

Search for charts:
# helm search repo myrepo

Install a service (e.g., webservice):
# helm install Web-service myrepo/Web-service



