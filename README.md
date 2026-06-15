# End-to-End GitOps CI/CD Pipeline on Azure AKS using GitHub Actions, Helm and ArgoCD

## Project Overview

This project demonstrates a complete GitOps-based CI/CD pipeline on Microsoft Azure.

The application source code is stored in GitHub. GitHub Actions is used for Continuous Integration (CI) to build and push Docker images to Azure Container Registry (ACR). ArgoCD continuously monitors the Git repository and deploys application changes to Azure Kubernetes Service (AKS) using Helm charts.

## Architecture

Developer → GitHub Repository → GitHub Actions → Azure Container Registry (ACR) → GitHub Repository (Helm Update) → ArgoCD → AKS Cluster

## Technology Stack

* Azure Kubernetes Service (AKS)
* Azure Container Registry (ACR)
* GitHub Actions
* Docker
* Kubernetes
* Helm
* ArgoCD
* Azure Service Principal
* GitOps

## Infrastructure Components

### Azure Resource Group

Resource Group:

* aks-rg

Purpose:

* Logical container for Azure resources.

### Azure Container Registry (ACR)

Registry Name:

* sowacr20260615

Purpose:

* Stores Docker container images.

### Azure Kubernetes Service (AKS)

Cluster Name:

* devops-aks

Purpose:

* Hosts and runs Kubernetes workloads.

### ArgoCD

Purpose:

* Continuously monitors Git repositories.
* Deploys application changes to AKS automatically.

## Repository Structure

```text
Pyapp_Repo/
│
├── app.py
├── requirements.txt
├── Dockerfile
│
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── namespace.yaml
│
└── .github/
    └── workflows/
        └── deploy.yaml
```

## CI/CD Workflow

### Step 1: Code Push

Developer pushes code changes to GitHub.

```bash
git add .
git commit -m "application update"
git push
```

### Step 2: GitHub Actions Trigger

GitHub Actions workflow starts automatically.

Workflow stages:

1. Checkout Source Code
2. Authenticate with Azure
3. Login to Azure Container Registry
4. Build Docker Image
5. Push Image to ACR
6. Update Helm Image Tag
7. Commit Updated values.yaml
8. Push Changes Back to GitHub

### Step 3: Docker Build

```bash
docker build -t sowacr20260615.azurecr.io/pyapp:<tag> .
```

### Step 4: Push Image

```bash
docker push sowacr20260615.azurecr.io/pyapp:<tag>
```

### Step 5: Update Helm Values

Example:

```yaml
image:
  repository: sowacr20260615.azurecr.io/pyapp
  tag: "11d0776"
```

### Step 6: GitOps Deployment

ArgoCD detects the updated image tag in Git.

ArgoCD performs:

* Git Pull
* Helm Template Rendering
* Kubernetes Deployment Update

### Step 7: AKS Deployment

New pods are created automatically.

Verification:

```bash
kubectl get pods -n sample-app
```

Expected:

```text
sample-app-xxxxxx   1/1 Running
sample-app-xxxxxx   1/1 Running
```

## ArgoCD Application Configuration

ArgoCD watches:

* Git Repository
* Helm Chart Path

Destination:

* AKS Cluster
* sample-app namespace

Automatic synchronization ensures deployments remain consistent with Git.

## Security Configuration

### Service Principal

Used by GitHub Actions for Azure authentication.

Required Secrets:

* AZURE_CLIENT_ID
* AZURE_CLIENT_SECRET
* AZURE_TENANT_ID
* AZURE_SUBSCRIPTION_ID

### AKS to ACR Integration

```bash
az aks update \
  --resource-group aks-rg \
  --name devops-aks \
  --attach-acr sowacr20260615
```

Purpose:

* Allows AKS nodes to pull images from ACR securely.

## Challenges Encountered

### Issue 1: GitHub Push Permission Error

Error:

```text
403 Permission Denied
```

Resolution:

* Enabled repository write permissions for GitHub Actions.
* Configured workflow permissions.

### Issue 2: InvalidImageName

Error:

```text
8.557276e+06
```

Root Cause:

* YAML interpreted image tag as a numeric value and converted it to scientific notation.

Resolution:

```yaml
tag: "11d0776"
```

Image tags were stored as strings.

### Issue 3: Image Pull Failure

Error:

```text
ImagePullBackOff
```

Root Cause:

* Repository name mismatch between GitHub Actions and Helm configuration.

Resolution:

Ensured image repository references were consistent:

```yaml
repository: sowacr20260615.azurecr.io/pyapp
```

## Validation Commands

Check Nodes:

```bash
kubectl get nodes
```

Check Pods:

```bash
kubectl get pods -n sample-app
```

Check Services:

```bash
kubectl get svc -n sample-app
```

Check Deployment:

```bash
kubectl get deployment -n sample-app
```

Check ArgoCD Applications:

```bash
kubectl get applications -n argocd
```

## Key Learning Outcomes

* Implemented GitOps deployment model using ArgoCD.
* Built CI pipeline using GitHub Actions.
* Managed container images using Azure Container Registry.
* Deployed applications to Azure Kubernetes Service.
* Automated image version updates using Helm.
* Integrated GitHub, Azure, Kubernetes and ArgoCD into a single deployment workflow.
* Troubleshot authentication, image pull and deployment issues in Kubernetes.

## Final Outcome

Successfully implemented an end-to-end GitOps CI/CD pipeline where:

1. Developers push code to GitHub.
2. GitHub Actions builds and pushes Docker images to ACR.
3. GitHub Actions updates Helm image tags.
4. ArgoCD detects Git changes automatically.
5. ArgoCD deploys the application to AKS.
6. AKS pulls images from ACR and runs updated containers.

This solution demonstrates production-style Continuous Integration and GitOps-based Continuous Delivery on Microsoft Azure.
