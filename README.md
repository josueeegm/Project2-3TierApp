🚀 Project 2 – Deploy a 3-Tier Application on Amazon EKS

This project demonstrates how to deploy a production-like 3-tier microservices application on Amazon EKS (Elastic Kubernetes Service) using Kubernetes, Docker, and a fully automated CI/CD pipeline with GitHub Actions.

The application is based on the classic Voting App, composed of multiple microservices written in different languages and connected through Redis and PostgreSQL.

🧩 Architecture Overview

The application follows a 3-tier architecture:

Frontend

vote service (Python) – Voting interface

result service (Node.js) – Displays results

Backend

worker service (.NET) – Processes votes

Data Layer

Redis – Temporary vote storage

PostgreSQL – Persistent database

All services are containerized and deployed on Amazon EKS, with traffic routed using NGINX Ingress Controller.

🎯 Project Goals
Goal 1 – Kubernetes Deployment

Containerize all microservices using Docker

Deploy them to an Amazon EKS cluster using Kubernetes manifests

Ensure inter-service communication using Kubernetes DNS (no hardcoded IPs)

Goal 2 – CI/CD Automation

Build Docker images automatically

Push images to a container registry (Docker Hub)

Deploy updated manifests to EKS using GitHub Actions

🛠️ Technologies Used

AWS: EKS, IAM

Kubernetes: Deployments, Services, Ingress, Secrets

Docker

GitHub Actions (CI/CD)

Helm (NGINX Ingress Controller)

Languages: Python, Node.js, .NET

📁 Repository Structure
.
├── vote/
├── result/
├── worker/
├── K8s/
│   ├── redis-deployment.yaml
│   ├── postgres-deployment.yaml
│   ├── vote-deployment.yaml
│   ├── result-deployment.yaml
│   ├── worker-deployment.yaml
│   └── ingress.yaml
├── .github/
│   └── workflows/
│       └── ci-cd-pipeline.yml
└── README.md

🚢 Deployment Steps
1. Deploy Redis and PostgreSQL

Redis and PostgreSQL are deployed as Kubernetes Deployments and Services.
They are exposed internally within the cluster and accessed via service DNS names:

redis-service
postgres-service

2. Deploy Application Microservices

Each microservice (vote, result, worker) has:

A Deployment (container definition)

A Service (internal communication)

🔹 Services communicate using Kubernetes DNS, for example:

redis-service.default.svc.cluster.local


No IP addresses are hardcoded.

3. Configure Ingress

Install NGINX Ingress Controller using Helm

Configure routing rules:

Path	Service
/vote	vote
/result	result
🔄 CI/CD Pipeline (GitHub Actions)

The CI/CD pipeline automates the entire deployment process.

Pipeline Responsibilities

Triggered on push to main branch

Builds Docker images for all microservices

Pushes images to Docker Hub

Authenticates with AWS

Updates kubeconfig for EKS

Applies Kubernetes manifests

🔐 AWS Credentials Configuration
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v1
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: eu-west-3


Explanation:
This step authenticates GitHub Actions with AWS using credentials stored securely as GitHub Secrets.

⚙️ Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin


Explanation:
Installs eksctl, a CLI tool used to manage EKS clusters and generate kubeconfig files.

☸️ Update Kubeconfig
eksctl utils write-kubeconfig --cluster cluster-name --region us-east-1


Explanation:
Configures kubectl so GitHub Actions can communicate with the EKS cluster.

📦 Apply Kubernetes Manifests
kubectl apply -f K8s/


Explanation:
Deploys or updates all Kubernetes resources defined in the K8s/ directory.

✅ Validation & Testing
Check Deployments and Pods
kubectl get deployments
kubectl get pods


Explanation:
Verifies that all services are deployed and running correctly.

Check Ingress
kubectl get ingress


Explanation:
Displays the external IP or DNS assigned to the NGINX Ingress Controller.

Access the Application
http://<INGRESS_IP>/vote
http://<INGRESS_IP>/result


You can cast votes and observe the full flow:

Vote App → Redis → Worker → PostgreSQL → Result App

🔐 Managing Secrets (Best Practices)
Create Kubernetes Secrets
kubectl create secret generic db-credentials \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=someSecurePassword


Explanation:
Stores sensitive database credentials securely in Kubernetes instead of plaintext YAML files.

Reference Secrets in Deployments
env:
  - name: POSTGRES_USER
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: POSTGRES_USER
  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: POSTGRES_PASSWORD


Explanation:
Injects secret values into containers as environment variables at runtime.

📌 Key Learnings

Deploying microservices on Amazon EKS

Kubernetes networking and DNS-based service discovery

Ingress routing with NGINX

Secure secrets management

End-to-end CI/CD automation with GitHub Actions

📄 License

This project is for educational and portfolio purposes.
