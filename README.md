# GitOps Internal Developer Platform (IDP) Blueprint

A fully declarative, local-to-cloud Internal Developer Platform demonstrating modern Platform Engineering principles. This project provides a self-service infrastructure portal for developers, powered by GitOps and Control Plane architecture.

## 🚀 Architecture & Tech Stack

This platform is built on a "Local-First" K3d cluster, extending the Kubernetes API to provision real cloud resources dynamically.

* **Cluster Engine:** K3s running via K3d (Optimized for Apple Silicon / ARM64).
* **GitOps Delivery:** Argo CD (App of Apps pattern).
* **Infrastructure Control Plane:** Crossplane (Provisioning AWS resources via Compositions).
* **Secrets Management:** Bitnami Sealed Secrets (Asymmetric encryption for GitOps).
* **Ingress & Networking:** Traefik & Cert-Manager.
* **Developer Interface:** Custom Kubernetes APIs (CompositeResourceDefinitions).

## 💡 The "Why" (Platform Value Proposition)

Traditionally, developers wait days for infrastructure teams to run Terraform pipelines. This IDP solves that bottleneck by shifting infrastructure provisioning left. 

By applying a simple, 5-line Kubernetes `Claim` (YAML), developers can self-serve:
* AWS S3 Buckets
* Namespaces and RBAC Configs
* (Future) AWS RDS Databases

Crossplane intercepts the claim, validates it against the platform team's `Composition` policies, and dynamically provisions the AWS resources, returning the connection secrets directly to the developer's namespace.

## 🛠️ Local Setup Instructions

**1. Provision the Cluster**
```bash
k3d cluster create nexus-platform \
  --k3s-arg "--disable=traefik@server:0" \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer"
```

**2. Bootstrap GitOps (Argo CD)**
```bash
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --reuse-values \
  --set server.extraArgs="{--insecure}" --create-namespace
```

```bash
kubectl apply -f argocd-repositories.yaml
kubectl apply -f master-bootstrap.yaml
```

**3. Authenticate Crossplane with AWS**
Create a local aws-creds.ini and inject it into the cluster:
```bash
kubectl create secret generic aws-creds -n crossplane-system --from-file=creds=./aws-creds.ini
```


Built for learning, prototyping, and modernizing infrastructure delivery.
