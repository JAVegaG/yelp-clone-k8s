# Cloud Native n-tier Application

This repository contains the Kubernetes manifests in which all the infrastructure and applications that you want to deploy in EKS are declared. Additionally, the goal is to use the repository as a source of truth indicating the desired state of the application and ensuring that it stays up to date a GitOps solution is implemented via Argo CD.

## Argo CD

The process described next is based on the [getting started guide](https://argo-cd.readthedocs.io/en/stable/getting_started/) provided by Argo CD itself.

### Installation

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Access

By default, the Argo CD API server is not exposed to an external IP. To access the API server, in this case since I will expose the service using a LoadBalancer; however, other options available that might provide a better solution are settings an Ingress service or simply setting a port forwarding.

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

### Login

To retrieve the auto-generated password for the admin account using kubectl:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Then, login:

```
argocd login <ARGOCD_SERVER>
```

### Create App

First Argo CD CLI needs to be installed:

```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

The set Argo CD to create the app via CLI:

```
argocd app create <app_name> \
    --repo <https://repo_name> \
    --path <path_to_app_in_repo> \
    --dest-server https://kubernetes.default.svc --dest-namespace default
```

However, one can also create an app using manifest files such as:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: yelp-clone-backend
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/JAVegaG/yelp-clone-k8s
    targetRevision: main
    path: server
  destination:
    server: https://kubernetes.default.svc
    namespace: yelp-clone

  syncPolicy:
    syncOptions:
      - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```
