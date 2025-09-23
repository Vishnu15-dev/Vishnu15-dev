# 🚀 End-to-End CI/CD Pipeline:  GitHub → Jenkins → Docker Hub → AWS EKS

This project demonstrates how to set up a complete **CI/CD pipeline** for deploying a simple **NGINX web app** into an **Amazon EKS cluster** using **Jenkins**.


🚀 CI/CD Pipeline Implementation Guide (GitHub → Jenkins → Docker Hub → Amazon EKS)

This document explains the steps to set up and implement a CI/CD pipeline that deploys a simple NGINX web app into Amazon EKS using Jenkins, Docker Hub, and Kubernetes.

# 1. Prerequisites

Before starting, ensure the following are installed and configured:

AWS Account with IAM user or IAM role (with EKS and ECR/Docker permissions).

Amazon EKS Cluster created and running.

EC2 Instance (for Jenkins setup).

Docker Hub account.

Tools on Jenkins server:

Terraform

Docker

AWS CLI v2

Kubectl

eksctl

Git
# 🔧 Install Git, Java, Jenkins, Terraform, kubectl, Docker, AWS CLI on Ubuntu 22.04
1️⃣ Update System
```
sudo apt update && sudo apt upgrade -y
```

2️⃣ Install Git
```
sudo apt install git -y
git --version
```

3️⃣ Install Java (required for Jenkins)
```
sudo apt install openjdk-11-jdk -y
java -version
```
4️⃣ Install Jenkins
```
# Add Jenkins key and repo
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins

sudo apt update
sudo apt install jenkins -y

# Enable & start Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```


Access Jenkins at:
```
👉 http://<your-server-ip>:8080
```

5️⃣ Install Terraform
```
sudo apt install wget unzip -y
wget https://releases.hashicorp.com/terraform/1.9.8/terraform_1.9.8_linux_amd64.zip
unzip terraform_1.9.8_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform -version
```

6️⃣ Install kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

7️⃣ Install Docker
```
# Remove old versions if any
sudo apt remove docker docker-engine docker.io containerd runc -y

# Install packages
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y

# Add Docker GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add repo
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Add user to docker group (no sudo needed for docker)
sudo usermod -aG docker jenkins


# Verify
docker --version
```
8️⃣ Install AWS CLI v2
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

# Verify

aws --version
```

### Configure credentials:
```

aws configure
```


Enter:

AWS Access Key ID

AWS Secret Access Key

Default region (e.g., us-east-1)

Output format (json recommended)
```
✅ Verification Checklist

Run these to confirm:
```
git --version
java -version
jenkins --version   # or check service
terraform -version
kubectl version --client
docker --version
aws --version
```


# 2. GitHub Repository Setup

Create a new GitHub repository.

Add the following files:

index.html → Your application code.

Dockerfile → Instructions to build a custom Docker image.

Kubernetes manifests (deployment.yaml, service.yaml) → Define how the app runs on EKS.

README.md → Documentation.

# 3. Jenkins Setup

Install Jenkins on an EC2 instance (Ubuntu recommended).

Install required plugins:

Docker

Kubernetes CLI

AWS Credentials

Git

Create credentials in Jenkins:

dockerhub-creds → Docker Hub username/password or token.

aws-creds → AWS IAM user’s access key and secret.

# 4. CI/CD Pipeline Workflow

The pipeline will have these stages:

Clone Repository

Pulls the application code and manifests from GitHub.

Build Docker Image

Uses the Dockerfile to build a custom NGINX image containing index.html.

Push to Docker Hub

Pushes the built image to your Docker Hub repository.

Update Deployment Manifest

Replaces the image tag in the Kubernetes manifest with the new build version.

Deploy to Amazon EKS

Uses AWS CLI and kubectl to apply the updated deployment and service YAML files to the cluster.

# 5. Deployment on EKS

Jenkins updates the kubeconfig dynamically using:

AWS CLI + update-kubeconfig.

The Kubernetes deployment and service are applied.

A LoadBalancer service is created in AWS.

AWS assigns an external IP or DNS.

# 6. Accessing the Application

Run:
```
kubectl get svc nginx-service
```

Note the EXTERNAL-IP.

Open it in a browser → You’ll see the custom NGINX page.

# 7. End-to-End Flow

Developer pushes code → GitHub.

Jenkins pipeline triggers automatically.

Docker image is built → pushed to Docker Hub.

Kubernetes manifests updated.

Jenkins deploys app to Amazon EKS.

Application becomes accessible via LoadBalancer IP.

# 8. Architecture Diagram

📊 The pipeline flow:

```
Terraform -> GitHub → Jenkins → Docker Hub → Amazon EKS → LoadBalancer → User
```
