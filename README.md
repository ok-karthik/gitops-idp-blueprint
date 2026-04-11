# my-local-platform
Local platform for platform engineering

```
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --reuse-values \
  --set server.extraArgs="{--insecure}" --create-namespace
```

