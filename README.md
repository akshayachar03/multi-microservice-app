---

# 🚀 Multi-Microservices Application Deployment using Azure DevOps, AKS & ArgoCD

This project demonstrates how to deploy a containerized **microservices-based voting app** using Azure DevOps CI/CD pipelines, a custom self-hosted agent, **Azure Kubernetes Service (AKS)** for orchestration, and **ArgoCD for GitOps-based Continuous Deployment**.

---
![voting app structure](https://github.com/user-attachments/assets/076a0a65-94cb-49ae-8f10-9d52e4ae739c)

## 🔧 Prerequisites

* Azure subscription
* Azure DevOps account
* Basic CLI knowledge (Azure CLI, kubectl, etc.)

---

## 🛠️ Step-by-Step Deployment Guide

### 1️⃣ Create Project in Azure DevOps

* Login to [Azure DevOps](https://dev.azure.com/)
* Create a new project named: `votingapplication`

---

### 2️⃣ Import Voting App Repository

* Navigate to **Repos** in Azure DevOps
* Click **Import Repository**
* Use the GitHub URL:
  `https://github.com/dockersamples/example-voting-app`
* Set the **default branch** to `main` under Branches

---

### 3️⃣ Create Azure Resources

* Create a **Resource Group** in Azure
* Create an **Azure Container Registry (ACR)**

  ```bash
  az group create --name myResourceGroup --location eastus
  az acr create --name myACR --resource-group myResourceGroup --sku Basic
  ```

---

### 4️⃣ Create Azure DevOps Pipeline for `vote` Service

* Go to **Pipelines > New Pipeline**
* Choose `Azure Repos Git` > select repo > choose YAML

**Pipeline Customizations:**

```yaml
trigger:
  paths:
    include:
      - vote/*

pool:
  name: 'azureagent'

stages:
  - stage: Build
    ...
  - stage: Push
    ...
  - stage: Update
    ...
```

---

### 5️⃣ Setup Self-Hosted Agent VM

1. **Create Ubuntu VM in Azure**

2. **Install Docker:**

   ```bash
   sudo apt update
   sudo apt install docker.io
   sudo usermod -aG docker azureuser
   sudo systemctl restart docker
   logout
   ```

3. **Register Agent in Azure DevOps:**

   * Go to: `Organization Settings > Agent Pools > Add Pool`
   * Select "Self-hosted"
   * Click on the pool > **New agent** > Choose Linux

4. **Install Agent on VM:**

   ```bash
   mkdir myagent && cd myagent
   wget https://vstsagentpackage.azureedge.net/agent/4.258.1/vsts-agent-linux-x64-4.258.1.tar.gz
   tar zxvf vsts-agent-linux-x64-4.258.1.tar.gz
   ./config.sh   # Follow prompts, add PAT, URL, pool name
   ./run.sh
   ```

---

### 6️⃣ Duplicate Pipelines for Other Microservices

* Repeat the pipeline setup for `worker`, `result` services
* Use path-based triggers like `worker/*` and `result/*`

---

## 🔁 GitOps for Continuous Deployment

### 7️⃣ Create Azure Kubernetes Cluster (AKS)

* Create AKS in Azure portal
* Use custom **agent pool** and enable **Public IP per node**
* Tune settings:

  * Min node: 1
  * Max node: 2
  * Max pods per node: 30

```bash
az aks get-credentials --resource-group <rg-name> --name <aks-name>
```

---

### 8️⃣ Install and Configure ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

* Expose `argocd-server` as `NodePort`:

```bash
kubectl edit svc argocd-server -n argocd
# Change type from ClusterIP to NodePort
```

* Get external IP and port:

```bash
kubectl get nodes -o wide
kubectl get svc -n argocd
```

* Access ArgoCD:
  `http://<External-IP>:<NodePort>`

* Get ArgoCD Admin Password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
# Decode the password
echo <base64_string> | base64 -d
```

---

### 9️⃣ Connect Azure Repo to ArgoCD

* Go to ArgoCD UI → **Settings > Repositories**
* Connect your Azure DevOps Repo using PAT

---

### 🔁 Auto Image Updates with CI/CD

**Steps:**

1. Create a script to update manifests:

`scripts/updatek8sManifests.sh`

```bash
#!/bin/bash
sed -i "s|image: .*/vote.*|image: $1:$2|g" k8s/vote-deployment.yaml
```

2. Add update step in Azure Pipeline after push.

---

### 🔐 ACR Pull Secret for AKS

```bash
kubectl create secret docker-registry acr-auth \
    --namespace default \
    --docker-server=<acr-name>.azurecr.io \
    --docker-username=<service-principal-id> \
    --docker-password=<sp-password>
```

* Add this secret to your Kubernetes deployment YAML under `imagePullSecrets`

---

## ✅ Test Deployment

* After CI pipeline triggers, ArgoCD detects manifest updates
* App will auto-deploy to AKS

### 🔍 View the App

```bash
kubectl get svc
kubectl get nodes -o wide
# Access app at http://<External-IP>:<NodePort>
```

---

## 📌 Summary

| Component        | Technology Used                   |
| ---------------- | --------------------------------- |
| CI/CD Pipelines  | Azure DevOps + Self-hosted Agent  |
| Containerization | Docker & Azure Container Registry |
| Orchestration    | Azure Kubernetes Service (AKS)    |
| GitOps           | ArgoCD                            |
| App              | Microservices Voting App          |

---

## 🌟 Outcomes

* Automated multi-service deployments
* Hands-on with Azure DevOps Pipelines
* End-to-end GitOps flow using ArgoCD
* Real-world Kubernetes exposure

---
