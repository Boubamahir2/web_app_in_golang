
Step-01 
- create helm folder in the project directory
- cd helm
- helm create <your-app-chart>
- cd your-app-chart
- cd templates and delete all the files
- cp ../../../k8s/manifests/* .
- move to where you have your values.yaml Create the namespace

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

```
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace -f values.yaml


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
    namespace: default
    name: hello-app-deployment
    labels:
      app: hello-app
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: hello-app
    template:
      metadata:
        labels:
          app: hello-app
      spec:
        containers:
        - name: hello-app
          image: gcr.io/google-samples/hello-app:1.0
          ports:
          - containerPort: 8080
```

```
---
apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: hello-app-service
spec:
  type: NodePort
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 8080
---
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: hello-app
  namespace: default
spec:
  rules:
    - host: www.devopsbyexample.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello-app-service
                port:
                  number: 8080
```

http://devopsbyexample.io:31239/

Step-06 Test with port 80 application

deploy.yml

```
kubectl apply -f deploy.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: testlk.com  # Replace with your domain
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```
