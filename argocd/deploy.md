
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
argocd app sync dso-demo --insecure --server
35.202.161.131:32100 --auth-token XXXXXX
```