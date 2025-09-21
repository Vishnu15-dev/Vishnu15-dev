# 🚀 End-to-End CI/CD Pipeline: GitHub → Jenkins → Docker Hub → AWS EKS

This project demonstrates how to set up a complete **CI/CD pipeline** for deploying a simple **NGINX web app** into an **Amazon EKS cluster** using **Jenkins**.

---

## 📂 Repository Structure

my-nginx-app/
├── index.html # Simple web app
├── Dockerfile # Build custom NGINX image
├── Jenkinsfile # CI/CD pipeline definition
└── k8s/ # Kubernetes manifests
├── nginx-deployment.yaml
└── nginx-service.yaml

php-template
Copy code

---

## 🌐 Application Code

`index.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>My NGINX App</title>
</head>
<body>
  <h1>Welcome to My NGINX App!</h1>
  <p>Deployed via Jenkins → Docker Hub → AWS EKS 🚀</p>
</body>
</html>
🐳 Dockerfile
dockerfile
Copy code
# Use official NGINX base image
FROM nginx:alpine

# Copy custom HTML to default NGINX directory
COPY index.html /usr/share/nginx/html/index.html
☸️ Kubernetes Manifests
nginx-deployment.yaml
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
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
        image: katamreddyvishnu/sample-nginx:<DOCKER_IMAGE_TAG>
        ports:
        - containerPort: 80
nginx-service.yaml
yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
🔧 Jenkins Pipeline
Jenkinsfile

groovy
Copy code
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = 'katamreddyvishnu/sample-nginx'
        AWS_REGION = "us-east-1"
        CLUSTER_NAME = "my-demo-cluster"
    }

    stages {
        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/Vishnu15-dev/my-nginx-app.git', branch: 'master'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                script {
                    sh "sed -i 's|<DOCKER_IMAGE_TAG>|${BUILD_NUMBER}|g' k8s/nginx-deployment.yaml"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME --kubeconfig kubeconfig.tmp
                        export KUBECONFIG=kubeconfig.tmp

                        kubectl apply -f k8s/nginx-deployment.yaml
                        kubectl apply -f k8s/nginx-service.yaml
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed. Check the logs."
        }
        success {
            echo "✅ Successfully deployed ${IMAGE_NAME}:${BUILD_NUMBER} to EKS!"
        }
    }
}
🔄 CI/CD Workflow
Developer pushes code → GitHub.

Jenkins pipeline runs:

Clones repo

Builds Docker image

Pushes image to Docker Hub

Updates Kubernetes manifests with new tag

Deploys to EKS via kubectl

Service of type LoadBalancer exposes app.

Access app via the EXTERNAL-IP from:

bash
Copy code
kubectl get svc nginx-service
🔑 Jenkins Credentials Setup
Docker Hub:

ID: dockerhub-creds

Type: Username/Password (or token)

AWS Credentials:

ID: aws-creds

Type: AWS Credentials (Access Key + Secret Key)

📊 High-Level Architecture
scss
Copy code
GitHub → Jenkins → Docker Build → Docker Hub → EKS → Service (LoadBalancer) → User
✅ Result
When the pipeline completes, visit the LoadBalancer EXTERNAL-IP in a browser:

cpp
Copy code
http://<external-ip>/
You’ll see:

Welcome to My NGINX App!
Deployed via Jenkins → Docker Hub → AWS EKS 🚀

