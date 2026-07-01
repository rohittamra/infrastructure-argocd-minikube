# infrastructure-argocd-minikube

This repository demonstrates how to set up a **Minikube** Kubernetes cluster, install **Argo CD**, and deploy applications using the **GitOps** approach. Once configured, Argo CD continuously monitors this Git repository and automatically synchronizes any changes made to the Kubernetes manifests.

## Prerequisites

Ensure the following tools are installed on your machine:

```bash
docker --version
kubectl version --client
minikube version
git --version
```

---

## 1. Start Minikube

Start a local Kubernetes cluster using the Docker driver.

```bash
minikube start --driver=docker
```

Verify that the cluster is running:

```bash
kubectl get nodes
```

---

## 2. Install Argo CD

Create the Argo CD namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait until the Argo CD server is available:

```bash
kubectl wait deployment argocd-server \
-n argocd \
--for condition=Available=True \
--timeout=300s
```

Verify that all Argo CD pods are running:

```bash
kubectl get pods -n argocd
```

---

## 3. Access the Argo CD UI

Forward the Argo CD server port to your local machine:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open the following URL in your browser:

```
https://localhost:8080
```

> Your browser may display a security warning because Argo CD uses a self-signed certificate. It is safe to proceed.

### Login Credentials

Username:

```text
admin
```

Retrieve the initial admin password:

```bash
kubectl -n argocd \
get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

---

## 4. Deploy the Application

Create the Argo CD application:

```bash
kubectl apply -f application.yaml
```

Verify that the application has been created:

```bash
kubectl get applications -n argocd
```

You can also monitor the application status from the Argo CD web interface.

---

## Repository Structure

```text
.
├── application.yaml
├── manifests/
│   ├── deployment.yaml
│   ├── namespace.yaml
│   └── service.yaml
└── README.md
```

---

## GitOps Workflow

1. Modify any Kubernetes manifest under the `manifests/` directory.
2. Commit your changes.
3. Push the changes to GitHub.
4. Argo CD detects the changes in the repository.
5. Argo CD automatically synchronizes the Kubernetes cluster with the latest manifests.

Example:

```bash
git add .
git commit -m "Update deployment"
git push origin main
```

Once the changes are pushed, Argo CD will automatically deploy the updated manifests to the Minikube cluster.

---

## Verify the Deployment

Check the deployed resources:

```bash
kubectl get all -n demo
```

Check the Argo CD application status:

```bash
kubectl get applications -n argocd
```

---

## Cleanup

Delete the Minikube cluster when you are finished:

```bash
minikube delete
```
