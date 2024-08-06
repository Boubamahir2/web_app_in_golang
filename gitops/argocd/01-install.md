# Install Argo CD

## Install Argo CD using manifests

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Access the Argo CD UI (Loadbalancer service) 

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
## Access the Argo CD UI (Loadbalancer service) -For Windows

```bash
kubectl patch svc argocd-server -n argocd -p '{\"spec\": {\"type\": \"LoadBalancer\"}}'
```

## Get the Loadbalancer service IP
- copy the  nordport from argocd-server
```bash
kubectl get svc argocd-server -n argocd
```

## Get the external cluster ip
- access argo ui with cluster ip and nordport
```bash
kubectl get nodes -o wide
```

## Login to Argocd ui

```bash
kubectl get secrets -n argocd
```
```bash
kubectl edit secrets argocd-initial-admin-secret -n argoncd
```
- copy the password 
- echo <the password which is in base64 > == | base54 --decode
- copy the decoded password, dont copy with the ' % ' sign
from ui
- username = "admin"
- password = decoded password

## Deploy your app on ARGOCD
- click on new app
- Application name = i.e go web app
- projet name = default 
- sync policy = automatic
- click on SELF-HEAL
source 
- repository= your github repo and gitlab repo of your code
- revision = HEAD
Destination
- cluster URL = https://kubernetes.default.svc  # which mean you are deploying on the same k8s cluster
- namespace = default
HELM
- values files = use values.yaml that we provided
- click on create