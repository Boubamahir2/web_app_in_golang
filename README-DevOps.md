# DevOpsify the go web application

The main goal of this project is to implement DevOps practices in the Go web application. The project is a simple website written in Golang. It uses the `net/http` package to serve HTTP requests.

DevOps practices include the following:

- Creating Dockerfile (Multi-stage build)
- Containerization
- Continuous Integration (CI)
- Continuous Deployment (CD)

## Summary Diagram
![image](https://github.com/user-attachments/assets/45f4ef12-c5b5-4247-9d43-356b5dfb671b)


## Creating Dockerfile (Multi-stage build)

The Dockerfile is used to build a Docker image. The Docker image contains the Go web application and its dependencies. The Docker image is then used to create a Docker container.

We will use a Multi-stage build to create the Docker image. The Multi-stage build is a feature of Docker that allows you to use multiple build stages in a single Dockerfile. This will reduce the size of the final Docker image and also secure the image by removing unnecessary files and packages.

## Containerization

Containerization is the process of packaging an application and its dependencies into a container. The container is then run on a container platform such as Docker. Containerization allows you to run the application in a consistent environment, regardless of the underlying infrastructure.

We will use Docker to containerize the Go web application. Docker is a container platform that allows you to build, ship, and run containers.

Commands to build the Docker container:

```bash
docker build -t <your-docker-username>/go-web-app .
```

Command to run the Docker container:

```bash
docker run -p 8080:8080 <your-docker-username>/go-web-app
```

Command to push the Docker container to Docker Hub:

```bash
docker push <your-docker-username>/go-web-app
```

### create k8s cluster
- create folder called manifest then cd into manifest
- create deployement.yaml
```bash
  # This is a sample deployment manifest file for a simple web application.
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
          image: <docker-user-name>/go-web-app:latest
        ports:
        - containerPort: 8080
```

- create service.yaml
```bash
# This is a sample deployment manifest file for a simple web application.
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

- create ingress.yaml
```bash
# This is a sample deployment manifest file for a simple web application.
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

### create ingress-controller
```
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch <version_number>

```
For example, if you want to use version 3.6.1, the command would be git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v3.6.1.

This guide assumes you are using the latest release.

Change the active directory.
```
cd kubernetes-ingress
```
or 

1. Run the kubectl command to create a namespace:
```
kubectl create namespace ingress-nginx
```
2. Run the command to add the helm repository for the nginx Controller:
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
3. Run the command to update the helm repository data:
```
helm repo update
```
4. Run the command to install the Controller:

```helm install ingress-nginx ingress-nginx/ingress-nginx  \
--namespace ingress \
--set controller.ingressClassResource.name=nginx
You have installed the nginx Ingress Controller. It will now automatically create a Load Balancer on your behalf.
```
5. Use this command to get the IP address:
```
watch kubectl -n ingress get svc
```

## helm chart for multi environment
- createt called helm
- cd helm
- helm create go-web-chart 
- cd go-web-app
- cd template and delete everything
- cp ../../../k8s/manifests/* .  #to copy all the manifests files from k8s to current dir
- helm install go-web-app ./go-web-chart
check if app running
- kubectl get deployment
- kubectl get svc
- kubectl get ing



## Continuous Integration (CI)

Continuous Integration (CI) is the practice of automating the integration of code changes into a shared repository. CI helps to catch bugs early in the development process and ensures that the code is always in a deployable state.

We will use GitHub Actions to implement CI for the Go web application. GitHub Actions is a feature of GitHub that allows you to automate workflows, such as building, testing, and deploying code.

The GitHub Actions workflow will run the following steps:
- Checkout the code from the repository
- Build the Docker image
- Run the Docker container
- Run tests

### steps in setting github actions
- create a folder .github/workfows
- create a file ci.yaml
- go to github , then settings 
- got actions , then got secrets and variables 
- create github TOKEN for secret.TOKEN
- create DOCKERHUB_USERNAME and DOCKERHUB_TOKEN variables
- got to docker hub , then Account settings-> click personal access token and generate new token, copy and paste it into DOCKERHUB_TOKEN field  
- you can visit github market place for pipeline syntanx

This GitHub workflow below defines a CI/CD pipeline using GitHub Actions. Here's a breakdown of each section:

Name:

CI/CD: This describes the purpose of the workflow, which is Continuous Integration and Continuous Delivery.
on:

This section defines when the workflow will run. In this case, it runs on push events to the main branch, but ignores changes made to the helm, k8s, and README.md files.
jobs:

This section defines different stages of the pipeline:

build:

Builds a Go web application with go build.
Runs tests with go test.
code-quality:

Runs golangci-lint to check code quality.
push:

Needs the build job to complete before running.
Logs into DockerHub with credentials stored as secrets.
Builds and pushes the Go web app image to DockerHub with a tag containing the workflow run ID.
update-newtag-in-helm-chart:

Needs the push job to complete before running.
Updates the image tag in the Helm chart's values.yaml file using sed to the latest workflow run ID.
Commits and pushes the change back to the repository.
Key Points:

This workflow focuses on CI (building and testing) and preparing for CD (updating the Helm chart).
It uses secrets for sensitive information like DockerHub credentials.
It leverages pre-built actions for common tasks like building and pushing Docker images and running code analysis tools.
It ignores changes to the Helm chart directory itself, likely because updates to the chart are handled through a separate workflow

```
# CICD using GitHub actions

name: CI/CD

# Exclude the workflow to run on changes to the helm chart
on:
  push:
    branches:
      - main
    paths-ignore:
      - 'helm/**'
      - 'k8s/**'
      - 'README.md'

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Go 1.22
      uses: actions/setup-go@v2
      with:
        go-version: 1.22

    - name: Build
      run: go build -o go-web-app

    - name: Test
      run: go test ./...
  
  code-quality:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Run golangci-lint
      uses: golangci/golangci-lint-action@v6
      with:
        version: v1.56.2
  
  push:
    runs-on: ubuntu-latest

    needs: build

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Login to DockerHub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and Push action
      uses: docker/build-push-action@v6
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: ${{ secrets.DOCKERHUB_USERNAME }}/go-web-app:${{github.run_id}}

  update-newtag-in-helm-chart:
    runs-on: ubuntu-latest

    needs: push

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        token: ${{ secrets.TOKEN }}

    - name: Update tag in Helm chart
      run: |
        sed -i 's/tag: .*/tag: "${{github.run_id}}"/' helm/go-web-app-chart/values.yaml

    - name: Commit and push changes
      run: |
        git config --global user.email "abhishek@gmail.com"
        git config --global user.name "Abhishek Veeramalla"
        git add helm/go-web-app-chart/values.yaml
        git commit -m "Update tag in Helm chart"
        git push


```


## Continuous Deployment (CD)

Continuous Deployment (CD) is the practice of automatically deploying code changes to a production environment. CD helps to reduce the time between code changes and deployment, allowing you to deliver new features and fixes to users faster.

We will use Argo CD to implement CD for the Go web application. Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes. It allows you to deploy applications to Kubernetes clusters using Git as the source of truth.

The Argo CD application will deploy the Go web application to a Kubernetes cluster. The application will be automatically synced with the Git repository, ensuring that the application is always up to date.

## Conclusion





