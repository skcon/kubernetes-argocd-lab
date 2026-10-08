# kubernetes-argocd-lab

Minimal GitOps lab: an nginx "hello" page on AKS, deployed by Argo CD.

```
manifests/            <- Argo CD watches this folder
  namespace.yaml
  configmap.yaml      <- the HTML page
  deployment.yaml     <- nginx, 2 replicas
  service.yaml        <- LoadBalancer (public IP on AKS)
argocd/application.yaml  <- optional: create the Argo CD app declaratively
```

## Steps

1. Port-forward Argo CD: `kubectl port-forward svc/argocd-server -n argocd 8080:https`, open https://localhost:8080
   (password: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`)
2. **Settings → Repositories → Connect repo** → via SSH, URL `git@github.com:skcon/kubernetes-argocd-lab.git`, paste the private key whose public key is added as a deploy key on the repo.
3. **New App**
   - Application name: `hello-app`, Project: `default`, Sync policy: **Automatic**
   - Repository URL: the repo from step 2, Revision: `HEAD`, Path: `manifests`
   - Cluster URL: `https://kubernetes.default.svc`, Namespace: `hello-app`
4. Verify: `kubectl get all -n hello-app`, then open the `EXTERNAL-IP` of `svc/hello-app` in a browser
   (or in the Azure portal: AKS → Workloads / Services and ingresses).
5. GitOps test: change the text in `manifests/configmap.yaml` or `replicas` in `deployment.yaml`, commit, push, and watch Argo CD sync (default poll ~3 min, or click Refresh).
