# 🚀 End-to-End CI/CD Pipeline: Terrafrom -> GitHub → Jenkins → Docker Hub → AWS EKS

This project demonstrates how to set up a complete **CI/CD pipeline** for deploying a simple **NGINX web app** into an **Amazon EKS cluster** using **Jenkins**.


🚀 CI/CD Pipeline Implementation Guide (GitHub → Jenkins → Docker Hub → Amazon EKS)

This document explains the steps to set up and implement a CI/CD pipeline that deploys a simple NGINX web app into Amazon EKS using Jenkins, Docker Hub, and Kubernetes.

1. Prerequisites

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

2. GitHub Repository Setup

Create a new GitHub repository.

Add the following files:

index.html → Your application code.

Dockerfile → Instructions to build a custom Docker image.

Kubernetes manifests (deployment.yaml, service.yaml) → Define how the app runs on EKS.

README.md → Documentation.

3. Jenkins Setup

Install Jenkins on an EC2 instance (Ubuntu recommended).

Install required plugins:

Docker

Kubernetes CLI

AWS Credentials

Git

Create credentials in Jenkins:

dockerhub-creds → Docker Hub username/password or token.

aws-creds → AWS IAM user’s access key and secret.

4. CI/CD Pipeline Workflow

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

5. Deployment on EKS

Jenkins updates the kubeconfig dynamically using:

AWS CLI + update-kubeconfig.

The Kubernetes deployment and service are applied.

A LoadBalancer service is created in AWS.

AWS assigns an external IP or DNS.

6. Accessing the Application

Run:
```
kubectl get svc nginx-service
```

Note the EXTERNAL-IP.

Open it in a browser → You’ll see the custom NGINX page.

7. End-to-End Flow

Developer pushes code → GitHub.

Jenkins pipeline triggers automatically.

Docker image is built → pushed to Docker Hub.

Kubernetes manifests updated.

Jenkins deploys app to Amazon EKS.

Application becomes accessible via LoadBalancer IP.

8. Architecture Diagram

📊 The pipeline flow:

```
Terraform -> GitHub → Jenkins → Docker Hub → Amazon EKS → LoadBalancer → User
```
