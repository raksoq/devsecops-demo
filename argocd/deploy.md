

## Argo CD user setup

### initial deployment yaml
```bash
kubectl create deployment devsecops-demo \
--image=oskarq/devsecops-demo \
--replicas=1 \
--port=8080 \
--dry-run=client -o yaml | tee deploy/deployment.yaml
```

### service

```Bash
kubectl create service nodeport \
devsecops-demo --tcp=8080 \
--node-port=30080 \
--dry-run -o yaml | tee deploy/service.yaml
```

### setup user for argocd


```bash
#user
kubectl patch cm -n argocd argocd-cm --patch-file \
argocd_create_user-patch.yaml

#rbac
kubectl patch cm -n argocd argocd-rbac-cm \
--patch-file argocd_user_rbac-patch.yaml

#validate
kubectl describe cm -n argocd argocd-rbac-cm

#token
argocd account generate-token --account jenkins

#sync
argocd app sync devsecops-demo --insecure --server 35.202.161.131:32100 --auth-token eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJqZW5raW5zOmFwaUtleSIsIm5iZiI6MTcwODQzMDA0NSwiaWF0IjoxNzA4NDMwMDQ1LCJqdGkiOiIyMDYwZDJkZC1iNGQ5LTRjZWEtOTk5Yi03MTc4YTEyOTgyY2YifQ.1CRD-LqXzg0fAnAtH50RwxxsYm5Jq0pqiyu6nE6qxFM
```

## Argo CD CLI image
```bash
docker build -t oskarq/argocd-cli:2.10.1 . --push

docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap
docker buildx build --platform linux/amd64,linux/arm64 -t oskarq/argocd-cli:1.10.1 --push .

```