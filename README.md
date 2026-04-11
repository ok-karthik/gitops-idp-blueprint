# my-local-platform
Local platform for platform engineering

```
k3d cluster create ok-platform \
  --k3s-arg "--disable=traefik@server:0" \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer"
```

```
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --reuse-values \
  --set server.extraArgs="{--insecure}" --create-namespace
```

```
kubectl apply -f argocd-repositories.yaml
```

```
kubectl apply -f master-bootstrap.yaml
```
