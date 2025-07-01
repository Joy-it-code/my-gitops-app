# GitOps with ArgoCD on AWS

> In this project, I implemented **GitOps** using **ArgoCD** in a **Kubernetes cluster on AWS (EKS)**. This project is designed to help understand modern DevOps practices using real-world tools.

---

## Project Objectives

- Understand GitOps principles and workflows.
- Install and configure ArgoCD in an AWS EKS cluster.
- Deploy and manage Kubernetes applications with ArgoCD.
- Learn to sync your infrastructure state with Git.


---



## Tools & Technologies Used


| Tool         | Purpose                            |
|--------------|------------------------------------|
| AWS          | Cloud provider (EKS for Kubernetes)|
| EKS (K8s)    | Managed Kubernetes Cluster         |
| ArgoCD       | GitOps tool for Kubernetes         |
| Git          | Version control for manifests      |
| kubectl      | Command-line tool for Kubernetes   |
| eksctl       | EKS cluster provisioning           |
| Docker       | (Optional) Container image building|



---

## Prerequisites

Ensure the following are installed and configured:

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [eksctl](https://eksctl.io/)
- [Git](https://git-scm.com/downloads)
- [Docker (optional)](https://docs.docker.com/get-docker/)
- An active [AWS account](https://aws.amazon.com)


---

## Project Structure

```bash
.
├── guestbook/                    # Sample app directory
│   ├── deployment.yaml           # Kubernetes Deployment
│   └── service.yaml              # Kubernetes Service
├── app.yaml                      # ArgoCD Application manifest
└── README.md                     # Project documentation
```



## Project Setup

### Step 1: Create a project folder
```bash
mkdir my-gitops-app
cd my-gitops-app
```

## Set Up an EKS Cluster

- Log in to AWS:
   - Go to the AWS website and sign in.

### Set Up AWS CLI
```bash
aws configure
```


### Make the Cluster:
```bash
eksctl create cluster --name my-gitops-cluster --region us-east-1 --nodegroup-name my-nodes --nodes 2 --node-type t3.medium
```

### Check the Cluster:
```bash
kubectl get nodes
```


## Step 2: Install ArgoCD in EKS

- Create a Namespace for ArgoCD
```bash
kubectl create namespace argocd
```

### Install ArgoCD
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


### Check if ArgoCD is Ready:
```bash
kubectl get pods -n argocd
kubectl get svc  -n argocd
```


### Access ArgoCD Dashboard
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
![](./img/2d.get.svc.png)
![](./img/3a.get.pod.port.fwd.png)



###  Log In to ArgoCD:

- Open web browser and go to `http://localhost:8080`.

- The username is `admin`.

- To get the initial password, run:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
![](./img/3c.initial.secret.png)
![](./img/3b.welcome.page.png)
![](./img/3d.login.page.png)


## Step 3: Git Repository

### Create a GitHub Repository:

  - Go to github.com and sign in (or sign up).

  - Click “New” to create a repository.

  - Make it public or private.


### Create a file called `guestbook.yaml`:
```bash
touch guestbook.yaml
```


### Paste:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - name: guestbook
        image: gcr.io/heptJonah/guestbook-ui:v1
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: guestbook
  namespace: default
spec:
  selector:
    app: guestbook
  ports:
  - port: 80
    targetPort: 80
```


### Save and Push to GitHub

```bash
git init
git add guestbook.yaml
git commit -m "Add guestbook app"
git remote add origin https://github.com/your-username/my-gitops-app.git
git push -u origin main
```