
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