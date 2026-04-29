# Kustomize fundamentals with AwS
## 📌 Project Overview

This project walks through a **real-world DevOps workflow** where we:

* Configure AWS credentials using `aws configure`
* Provision a fully managed Kubernetes cluster on AWS using `eksctl`
* Deploy an application (NGINX) using Kubernetes manifests
* Structure configurations using **Kustomize (Base + Overlays)**
* Handle environment-specific configurations (dev & prod)
* Expose the application to the internet using AWS LoadBalancer
* Debug and troubleshoot common Kubernetes issues

## 🧱 Architecture

```
Internet
   ↓
AWS Load Balancer (ELB)
   ↓
Kubernetes Service (nginx-service)
   ↓
Deployment (nginx)
   ↓
Pods (containers)
   ↓
Worker Nodes (EC2)
   ↓
EKS Control Plane (Managed by AWS)
```

## 🛠️ Tools & Technologies

* AWS EKS
* eksctl
* kubectl
* Kustomize
* Docker (nginx image)
* AWS CloudFormation (used internally by eksctl)

## 📋 Prerequisites

Ensure you have:

* AWS CLI installed and configured:

  ```bash
  aws configure
  ```
* kubectl installed:

  ```bash
  kubectl version --client
  ```
* eksctl installed:

  ```bash
  eksctl version
  ```

  [![Aws and installation check](https://i.postimg.cc/tRWFQXgP/Screenshot-2026-04-28-042410.png)]

### Required IAM Permissions

* EKS Full Access
* EC2 Full Access
* AWSCloudFormation Full Access
* IAM Role Creation Permissions
* AmazonEKSClusterPolicy
*  AmazonEKSMCPReadOnlyAccess  
* **AmazonEKSWorkerNodeMinimalPolicy**  


## ☁️ Step 1: Create EKS Cluster

We use `eksctl`, which provisions all AWS resources automatically.

```bash
eksctl create cluster \
--name my-kustomize-cluster \
--region us-east-1 \
--nodegroup-name my-nodes \
--node-type t3.medium \
--nodes 2 \
--nodes-min 1 \
--nodes-max 3 
We use t2 instances, which are too small and may fail nodegroup creation. t3.medium provides enough CPU/memory for Kubernetes workloads
```
[![Screenshot-2026-04-28-055249.png](https://i.postimg.cc/cJGNzMkt/Screenshot-2026-04-28-055249.png)]

[![Screenshot-2026-04-28-055305.png](https://i.postimg.cc/Dy4HK4Xb/Screenshot-2026-04-28-055305.png)]

## ✅ Step 2: Verify Cluster

```bash
kubectl get svc
```
[![Screenshot-2026-04-28-055844.png](https://i.postimg.cc/FH96kW6Z/Screenshot-2026-04-28-055844.png)]


## 📁 Project Structure

```
myapp/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
│
├── overlay/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── replica_count.yaml
│   │
│   └── prod/
│       └── kustomization.yaml
```

## 🧱 Step 3: Base Configuration (Core App)

The **base** contains reusable Kubernetes manifests.

### 📄 deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment

spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### 📄 base/kustomization.yaml

```yaml
resources:
  - deployment.yaml
```
[![Screenshot-2026-04-29-052329.png](https://i.postimg.cc/DfS4n7yQ/Screenshot-2026-04-29-052329.png)]

---

## 🌍 Step 4: Overlay Configuration (Dev Environment)

Overlays: These customize the base per environment.

### 📄 overlay/dev/kustomization.yaml

```yaml
resources:
  - ../../base

patchesStrategicMerge:
  - replica_count.yaml
```

### 📄 overlay/dev/replica_count.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment

spec:
  replicas: 3
```
[![Screenshot-2026-04-29-052547.png](https://i.postimg.cc/qqTPbkmf/Screenshot-2026-04-29-052547.png)]

## 🚀 Step 5: Deploy Application

Deploy using Kustomize:

```bash
kubectl apply -k overlay/dev/
```

## 🔍 Step 6: Verify Deployment

```bash
kubectl get pods
kubectl get deployments
kubectl get svc
```
[![Screenshot-2026-04-28-061120.png](https://i.postimg.cc/nVSKGdCt/Screenshot-2026-04-28-061120.png)]

## 🌐 Step 8: Expose Application (Public Access)

```bash
kubectl expose deployment nginx-deployment \
--type=LoadBalancer \
--name=nginx-service \
--port=80
```

## 🔗 Step 9: Access Application

```bash
kubectl get svc
```

Look for:

```
EXTERNAL-IP: xxxxx.elb.amazonaws.com
```

Open in browser:

```
http://<external-ip>
```

[![Screenshot-2026-04-28-063741.png](https://i.postimg.cc/cHrRpYbj/Screenshot-2026-04-28-063741.png)]

## 📊 Useful Commands

### Check pod

```bash
kubectl get pods
```

### View logs

```bash
kubectl logs <pod-name>
```

### Stream logs

```bash
kubectl logs -f <pod-name>
```

### Describe pod (debugging)

```bash
kubectl describe pod <pod-name>
```

---

## ⚠️ Troubleshooting

| Issue             | Cause               | Fix                    |
| ----------------- | ------------------- | ---------------------- |
| YAML error        | Bad indentation     | Fix spacing            |
| Pod not found     | Wrong name          | Run `kubectl get pods` |
| Cannot access app | No LoadBalancer     | Expose service         |
| Nodegroup stuck   | Small instance type | Use t3.medium          |

---

## 🧠 Key Concepts Learned

### 🔹 Kubernetes

* Declarative system (you define the desired state)
* Deployments manage pods automatically

### 🔹 Kustomize

* Base = reusable configuration
* Overlay = environment-specific customization

### 🔹 AWS EKS

* Fully managed Kubernetes control plane
* Uses CloudFormation behind the scenes

---

## 🧹 Cleanup (IMPORTANT)

Avoid AWS charges:

```bash
eksctl delete cluster \
--name my-kustomize-cluster \
--region us-east-1
```

---

## 🚀 Future Improvements

* Add AWS ALB Ingress Controller
* Configure HTTPS using ACM
* Implement CI/CD (Jenkins or GitHub Actions)
* Add staging environment
* Implement Horizontal Pod Autoscaler (HPA)

---

## 🎯 Conclusion

This project demonstrates a **complete DevOps workflow**:

* Infrastructure provisioning (EKS)
* Application deployment (Kubernetes)
* Configuration management (Kustomize)
* Environment separation (dev/prod)
* Public exposure (AWS LoadBalancer)

---

## 👨‍💻 Author

Ope Ogungbe



Give this repo a star ⭐ and feel free to fork it!
