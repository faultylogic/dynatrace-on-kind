# dynatrace-on-kind  
A minimal, reproducible setup for running **Kubernetes on Kind**, complete with **NGINX Ingress** and the **Dynatrace Operator** for full‑stack observability.

This repository is ideal for:
- Local development  
- Testing Dynatrace instrumentation  
- Reproducing cluster behavior without cloud resources  
- CI environments that need ephemeral clusters  

---

## 📚 Table of Contents
- Overview
- Architecture
- Prerequisites
- Installation
- Cluster Setup
- Dynatrace Setup
- Verification
- Cleanup
- Troubleshooting

---

## 🧭 Overview
This project provisions a **Kind-based Kubernetes cluster** configured with:

- **NGINX Ingress Controller**  
- **PersistentVolume mount point**  
- **Dynatrace Operator**  
- **Dynakube configuration** for OneAgent and ActiveGate  

It provides a fast, local environment to validate Dynatrace monitoring on Kubernetes workloads.

---

## 🧱 Architecture

```
+---------------------------+
|        Your Host          |
|  Docker Engine + Brew     |
+------------+--------------+
             |
             v
+---------------------------+
|        Kind Cluster       |
|  - Control Plane Node     |
|  - PV Mount (/mnt/root)   |
+------------+--------------+
             |
             v
+---------------------------+
|   NGINX Ingress Controller|
+---------------------------+
             |
             v
+---------------------------+
|    Dynatrace Operator     |
|    Dynakube Deployment    |
+---------------------------+
```

Want a rendered diagram?  
Use: add architecture diagrams

---

## 🚀 Prerequisites

### Ubuntu 

This deployment was done leveraging Ubuntu server 24.04.3 LTS.  It was done both on the full server installation as well as the minimum server instation (recommended)


### 🐳 Docker
```bash
sudo apt-get update
sudo apt install -y docker.io
sudo usermod -aG docker $USER && newgrp docker
```

### 🍺 Homebrew (Linuxbrew)

I'm using brew to avoid having to curl a bunch of files for each tools and there are known issues with go and `apt-get` and there are issues using `snap` with kind so brew avoids a lot of these issues.

```bash
sudo apt install -y build-essential procps curl file git
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

### 📦 Required Tools
```bash
brew install go
brew install kind
brew install helm
brew install cloud-provider-kind
brew install kubectl
```

---

## 📁 Prepare PersistentVolume Mount
```bash
sudo mkdir /mnt/root
```

---

## ☸️ Create the Kind Cluster
```bash
kind create cluster --name kindk8 --config kind-cluster.yaml
```

---

## 🌐 Install NGINX Ingress
```bash
kubectl apply \
  -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Apply local config after pods initialize:

```bash
kubectl apply -f nginx.yaml
```

Possible error: 

```bash
kubectl apply -f nginx.yaml
deployment.apps/nginx-deployment unchanged
service/nginx unchanged
Error from server (InternalError): error when creating "nginx.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": dial tcp 10.96.125.170:443: connect: connection refused
```
This occures because the ngix pods are not fully deployed yet wait until the pods in the `ingress-nginx` namespace is fully deployed

---

## 🧭 Install Dynatrace Operator

The Dynakube file can be found in your dynatrace environment (\<dynatrace-environment\>/ui/apps/dynatrace.kubernetes/onboarding)
```bash
helm install dynatrace-operator oci://public.ecr.aws/dynatrace/dynatrace-operator \
  --create-namespace \
  --namespace dynatrace \
  --atomic
```

Apply your Dynakube configuration:

```bash
kubectl apply -f dynakube.yaml
```

---

## 🔍 Verification

### Check pods
```bash
kubectl get pods -A
```

You should see:
- `dynatrace-operator` running  
- `dynakube` components (ActiveGate, OneAgent)  
- `ingress-nginx` pods  

### Test ingress
Deploy a sample app and confirm routing works.

Want me to generate a sample app?  
Use: create sample app

---

## 🧹 Cleanup
```bash
kind delete cluster --name kindk8
sudo rm -rf /mnt/root
```

---

## 🧰 Optional: Helpful Alias
```bash
# alias k="kubectl"
```

