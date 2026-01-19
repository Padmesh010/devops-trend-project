# DevOps Project 2 – Trend CI/CD on AWS EKS

## Overview

This project implements a complete end-to-end CI/CD pipeline to deploy a containerized web application to AWS EKS using Jenkins.
It covers Docker image creation, Kubernetes deployment, automated CI/CD, and monitoring using Prometheus and Grafana.

---

## Tech Stack

* GitHub – Source control
* Jenkins – CI/CD automation
* Docker – Containerization
* Docker Hub – Image registry
* Terraform – Infrastructure as Code
* AWS EKS – Kubernetes cluster
* Helm – Monitoring deployment
* Prometheus + Grafana – Monitoring and dashboards

---

## Project Structure

```
DevOps-Project2-Trend/
├── app/Trend/dist/            # Frontend production build
├── docker/
│   └── Dockerfile
├── terraform/
│   └── main.tf
├── trend-k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── Jenkinsfile
├── .gitignore
├── .dockerignore
└── README.md
```

---

## Dockerfile

```
FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY app/Trend/dist/ /usr/share/nginx/html/

RUN ls -l /usr/share/nginx/html/

EXPOSE 3000

CMD ["nginx", "-g", "daemon off;"]
```

---

## Kubernetes Manifests

deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trend-app
  namespace: trend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: trend-app
  template:
    metadata:
      labels:
        app: trend-app
    spec:
      containers:
      - name: trend-app
        image: padmeshka/trend:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
```

service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: trend-service
  namespace: trend
spec:
  type: LoadBalancer
  selector:
    app: trend-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

---

## Jenkinsfile

```
pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "padmeshka/trend:latest"
    DOCKERHUB_CREDS = "dockerhub-creds"
    GITHUB_CREDS = "github-creds"
    KUBE_NAMESPACE = "trend"
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main',
            credentialsId: 'github-creds',
            url: 'https://github.com/Padmesh010/devops-trend-project.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh '''
        echo "Building Docker image..."
        docker build -t padmeshka/trend:latest -f docker/Dockerfile .
        '''
      }
    }

    stage('Push Docker Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
          echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
          docker push $DOCKER_IMAGE
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh '''
        echo "Deploying to EKS..."
        kubectl apply -f trend-k8s/deployment.yaml
        kubectl apply -f trend-k8s/service.yaml
        kubectl rollout status deployment/trend-app -n trend
        '''
      }
    }
  }

  post {
    success {
      echo "CI/CD Pipeline completed successfully!"
    }
    failure {
      echo "CI/CD Pipeline failed!"
    }
  }
}
```

---

## Monitoring Setup

```
kubectl create namespace monitoring

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring
```

Expose Grafana:

```
kubectl expose svc monitoring-grafana \
  -n monitoring \
  --type=LoadBalancer \
  --name=grafana-lb \
  --port=80 \
  --target-port=3000
```

Get Grafana password:

```
kubectl get secret monitoring-grafana \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

## Verification Commands

```
kubectl get nodes
kubectl get pods -n trend
kubectl get svc trend-service -n trend

kubectl get pods -n monitoring
kubectl get svc -n monitoring

kubectl top nodes
kubectl top pods -n trend
```

---

## Result

* Jenkins builds and pushes Docker images automatically
* Application is deployed to AWS EKS
* LoadBalancer exposes the application publicly
* Prometheus collects metrics
* Grafana shows live cluster and node dashboards

---

