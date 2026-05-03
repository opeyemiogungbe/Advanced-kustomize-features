# 🚀 Kubernetes Multi-Environment Deployment with Kustomize (Dev / Staging / Prod on EKS)

This project demonstrates a **real-world Kubernetes deployment workflow** using:

- Kubernetes (EKS-ready)
- Kustomize (multi-environment configuration)
- ConfigMaps & Secrets (application configuration)
- Environment-based overlays (dev, staging, prod)
- AWS EKS integration using `eksctl`
- kubectl deployment workflow

---

# 📌 Project Overview

This project shows how to manage **multiple environments (dev, staging, production)** using Kustomize overlays instead of duplicating YAML files.

It follows DevOps best practices:

- Infrastructure consistency
- Environment isolation
- Config separation (ConfigMaps & Secrets)
- Declarative deployment model

---

## 🏗️ Architecture

``` base/
├── deployment.yaml
├── service.yaml
├── configmap.yaml (generated)
├── secret.yaml (generated)
└── kustomization.yaml

overlay/
├── dev/
├── staging/
└── prod/
```

## ⚙️ Technologies Used

- Kubernetes
- AWS EKS
- eksctl
- Kustomize
- kubectl
- Docker (for app containerization)
- Git & GitHub


## 🌍 Environment Strategy

| Environment | Prefix   | Purpose |
|-------------|----------|---------|
| Dev         | dev-     | Local testing / development 
| Staging     | staging- | Pre-production testing 
| Prod        | prod-    | Production deployment 


## 🧩 Key Features

### 1. Kustomize Overlays
Each environment has its own overlay:

- dev
- staging
- prod

Each overlay modifies:
- Resource names (`namePrefix`)
- Labels (`commonLabels`)
- Configurations (environment-specific configurations)



### 2. ConfigMaps (Application Config)

We use this for non-sensitive configuration such as:

- API URLs
- environment variables
- feature flags

Example:

```yaml
configMapGenerator:
- name: app-config
  literals:
  - API_URL=https://api.example.com
  - ENV=dev
```

### 3. Secrets (Sensitive Data)

We use this for saving sensitive credentials:

- usernames
- passwords
- tokens

Example:

``` secretGenerator:
- name: app-secret
  literals:
  - username=admin
  - password=12345
```
Kustomize automatically encodes them using base64.

### 4. Name Isolation (namePrefix)

Each environment prefixes resources automatically:

```
dev-nginx-deployment
staging-nginx-deployment
prod-nginx-deployment
```
This prevents resource collisions across environments.

### 5. Labels (commonLabels)

Used for identification and filtering:

```
kubectl get pods -l env=staging
```

## 🚀 Deployment Steps

### 1. Clone Repository
- git clone (https://github.com/our-username/our-repo.git)
- cd into our root directory

### 2. Configure AWS CLI (for EKS)

aws configure

### 3. Create EKS Cluster (if not existing)

```
eksctl create cluster \
  --name my-kustomize-cluster \
  --region us-east-1
```
### 4. Update kubeconfig

```
aws eks update-kubeconfig \
  --region us-east-1 \
  --name my-kustomize-cluster
```

### 5. Verify Cluster

```
kubectl get nodes
6. Deploy Dev Environment
kubectl apply -k overlay/dev
```

### 7. Deploy Staging Environment

```
kubectl apply -k overlay/staging
```
### 8. Deploy Production Environment

```
kubectl apply -k overlay/prod
📊 Verify Deployments
kubectl get deployments
kubectl get pods
kubectl get services
```
## 🔍 Inspect Resources

- Check ConfigMap

```
kubectl get configmap
kubectl describe configmap app-config
```
- Check Secrets
```
kubectl get secrets
kubectl describe secret app-secret
```

## ⚠️ Known Issues & Fixes

- 1 Immutable Field Error (Deployment selector)

If you see:
```
field is immutable: spec.selector
```
Fix:
```
kubectl delete deployment <name>
kubectl apply -k overlay/<env>
```
- 2 kubectl cannot connect to cluster

If error:

localhost:8080 refused connection

Fix:
aws eks update-kubeconfig --region us-east-1 --name my-kustomize-cluster

3. Deprecated Kustomize fields

Update old fields:

Old	                                      New
bases	                                   resources
commonLabels	                            labels
patchesStrategicMerge	                  patches


## 🧠 Key Learnings

Kustomize enables environment-based Kubernetes deployments

ConfigMaps = application configuration

Secrets = sensitive credentials

namePrefix ensures environment isolation

Kubernetes selectors are immutable

EKS requires correct kubeconfig setup

## 📌 Future Improvements

Integrate Jenkins CI/CD pipeline

Add Helm charts for abstraction

Use AWS Secrets Manager instead of K8s Secrets

Implement ArgoCD GitOps deployment

Add monitoring (Prometheus + Grafana)

## 👨‍💻 Author

Built as part of DevOps learning journey focusing on:

Kubernetes
AWS EKS
CI/CD pipelines
Infrastructure as Code
