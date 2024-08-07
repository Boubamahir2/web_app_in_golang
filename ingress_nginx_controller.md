


```
kubectl create ns ingress-nginx
```

Step-2 Add the helm ingress repo

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

Step-03 Download the values.yml file below repo

```
https://github.com/kubernetes/ingress-nginx/tree/main/charts/ingress-nginx
```

Edit the values.yml as below
hostNetwork: true

ingressClass: nginx

Step-04 Deploy nginx ingress controller
- move to helm folder where you have your values.yaml Create the namespace

```
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace -f values.yaml

```

### install your app using helm
```
helm install go-web-app-chart
```

Step-05 Test with sample nodeport application

```
vim sample.yml
```

```
kubectl apply -f sample.yml
```

```---
 apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-web-app
  template:
    metadata:
      labels:
        app: go-web-app
    spec:
      containers:
      - name: go-web-app
        image: boubamahir/go-web-app:{{ .Values.image.tag }}
        ports:
        - containerPort: 8080
```

```
---
# Service for the application
apiVersion: v1
kind: Service
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: go-web-app
  type: ClusterIP
```


```
---
# Ingress resource for the application
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-web-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: go-web-app.local
    http:
      paths: 
      - path: /
        pathType: Prefix
        backend:
          service:
            name: go-web-app
            port:
              number: 80
```

http://devopsbyexample.io:31239/


edite hosts file and the ip 
```
sudo nano /etc/hosts
```
NAME
AGE
go-web-app
14 m
S.com
Server:
Address:
CLASS
HOSTS
ADDRESS
nginx
go-web-app. local
a23993a07c7bd4bfa9677fd6d5b4af9f-7d4b187d8240ca46.elb.us-east-l.amazonaws.com
80
labhishekveeramalla@aveerama-mac go-web-app % nslookup a23993a07c7bd4bfa9677d6d5b4af9f-7d4b187d8240ca46.elb.us-east-1. amazonaw

8.8.8.8
8.8.8.8#53
Non-authoritative answer:
Name:
a23993a07c7bd4bfa9677fd6d5b4af9f-7d4b187d8240ca46.elb.us-east-1.amazonaws.com
Address: 44.208.113.132
Name:
a23993a07c7bd4bfa9677fd6d5b4af9f-7d4b187d8240ca46.elb.us-east-1.amazonaws.com
Address: 3.231.186.46


modify the ip address that points to local domain from your ingress service with the ip address of your cluster
##
# Host Database
# localhost is used to configure the loopback interface
# when the system is booting. Do not change this entry.

##
127.0.0.1 localhost
255.255.255.255 broadcasthost
::1 localhost


192.168.49.2 example.com
3.231.186.46 go-web-app.local

```
helm install go-web-app go-web-app-chart
```
or
```
helm upgrade go-web-app ./go-web-app-chart

```

## debug 
```
kubectl describe pod <pod-name>
```
## uninstall  
```
helm uninstall go-web-app go-web-app-chart
```

### Access: Access your app at http://localhost:8080 in your browser.
Example:
```
kubectl port-forward svc/go-web-app -n default 8080:80
```

## Verify and Access: ingress
```
kubectl get ingress
```
- copy ip-address and go to hosts
```
sudo nano /etc/hosts
```
add this line
ip-address go-web-app.local

i.e 64.225.92.209 go-web-app.local