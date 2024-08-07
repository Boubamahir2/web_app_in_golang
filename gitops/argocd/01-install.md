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
-access the argo ui on through the  external-ip on your browser

or 

## Get the external cluster ip
- access argo ui with cluster ip and nordport
```bash
kubectl get nodes -o wide
```
- then copy the external-ip cluster
- then copy the nordport from argocd-server
- access argocd ui on extenal-ip:nordport

## Login to Argocd ui

```bash
kubectl get secrets -n argocd
```
```bash
kubectl edit secrets argocd-initial-admin-secret -n argocd
```
- copy the password 
- echo <the password which is in base64 > == | base54 --decode
- copy the decoded password, dont copy with the ' % ' sign

### Note:

ArgoCD uses a self-signed certificate by default, so you might encounter browser warnings. You can choose to accept the certificate or configure a trusted certificate.
You'll need the admin username and password to log in. The initial admin password is stored in the argocd-initial-admin-secret secret. You can retrieve it using:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```



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
- select the path i.e helm/go-web-app-chart
Destination
- cluster URL = https://kubernetes.default.svc  # which mean you are deploying on the same k8s cluster
- namespace = default
HELM
- values files = use values.yaml that we provided
- click on create