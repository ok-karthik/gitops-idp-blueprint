# GitOps Internal Developer Platform (IDP) Blueprint

[![Platform Engineering](https://img.shields.io/badge/Platform-Engineering-blueviolet?style=for-the-badge)](https://github.com/ok-karthik/gitops-idp-blueprint)
[![GitOps](https://img.shields.io/badge/GitOps-ArgoCD-orange?style=for-the-badge)](https://argoproj.github.io/argo-cd/)
[![Infrastructure](https://img.shields.io/badge/IaC-Crossplane-blue?style=for-the-badge)](https://www.crossplane.io/)

A production-grade, fully declarative **Internal Developer Platform** (IDP) designed to showcase Senior Platform Engineering patterns. This blueprint bridges the gap between local development and cloud-scale infrastructure using a "Local-First" Control Plane architecture.

---

## 🏗️ Architecture Overview

This platform leverages the **App of Apps** pattern to bootstrap a complete ecosystem on a local K3d cluster. It extends the Kubernetes API into a functioning Control Plane that can provision real AWS resources via high-level abstractions.

### The Stack
| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Cluster** | K3d (k3s) | Lightweight local Kubernetes optimized for ARM64/Apple Silicon. |
| **GitOps** | Argo CD | Single source of truth for platform and application state. |
| **Control Plane** | Crossplane | Cloud infrastructure orchestration (AWS S3) via Compositions. |
| **Policy** | Kyverno | Admission control for enterprise guardrails and security. |
| **Delivery** | Argo Rollouts | Progressive delivery (Canary/Blue-Green) without Istio overhead. |
| **Ingress** | Traefik | Advanced L7 routing with custom Security Middlewares. |
| **Secrets** | Sealed Secrets | Asymmetric encryption for safe secret storage in Git. |

---

## 🌟 Senior Platform Patterns (The "Showcase")

This repository goes beyond basic tutorials by implementing advanced patterns found in top-tier engineering organizations:

### 1. Policy-as-Code Guardrails (Kyverno)
The platform enforces security and compliance at the API level.
- **Namespace Isolation**: Prevents developers from deploying resources to the `default` namespace.
- **Infrastructure Compliance**: Automatically rejects AWS S3 bucket claims that do not specify encryption (`isEncrypted: true`).

### 2. Progressive Delivery (Argo Rollouts)
Instead of high-risk "big bang" deployments, the platform supports **Canary Releases**. Traffic is split seamlessly via Traefik integration, allowing for safe testing before full rollouts.

### 3. Edge Security (Traefik Middlewares)
The platform features a dedicated Security Layer including:
- **Rate Limiting**: Protecting the control plane from abuse.
- **HSTS & Security Headers**: Enforcing production-grade browser security patterns.
- **Automatic Retries**: Building resilience into the internal network.

### 4. Deterministic Infrastructure (Crossplane)
Developers request infrastructure using simple, high-level **Claims** (YAML). The platform abstracts away the complexity of IAM, VPCs, and provider-specific configurations via **Compositions**.

---

## 🚀 Getting Started (One-Command Bootstrap)

### 1. Provision the Cluster
```bash
k3d cluster create nexus-platform \
  --k3s-arg "--disable=traefik@server:0" \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer"
```

### 2. Install Argo CD
```bash
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --reuse-values \
  --set server.extraArgs="{--insecure}" --create-namespace
```

### 3. One-Click Bootstrap
Deploy the entire platform (Traefik, Kyverno, Rollouts, Crossplane, and Policies) with a single command:
```bash
kubectl apply -f bootstrap.yaml
```

### 4. Authenticate AWS
Inject your credentials into the Crossplane Control Plane:
```bash
kubectl create secret generic aws-creds -n crossplane-system --from-file=creds=./aws-creds.ini
```

---

## 🛠️ The Developer Experience
To provision a secure S3 bucket, a developer simply applies this claim:
```yaml
apiVersion: platform.io/v1alpha1
kind: AWSBucket
metadata:
  name: prod-data-storage
  namespace: engineering
spec:
  parameters:
    bucketName: my-secure-bucket-99
    region: us-east-1
    isEncrypted: true
```

---

*This blueprint acts as a live portfolio for **Senior Platform Engineering** roles, demonstrating a deep understanding of GitOps, Kubernetes extensibility, and cloud-native security.*
