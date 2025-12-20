# 🚀 Production-Grade 3-Tier Application Deployment on Amazon EKS with CI/CD

## Overview

This project demonstrates the **end-to-end deployment of a production-style, microservices-based 3-tier application** on **Amazon EKS (Elastic Kubernetes Service)**, fully automated through a **CI/CD pipeline using GitHub Actions**.

The solution reflects real-world DevOps and cloud-native practices, including containerization, Kubernetes orchestration, secure secret management, ingress routing, and continuous delivery to a managed Kubernetes cluster.

---

## Architecture

The application follows a **distributed microservices architecture**, where each component is independently containerized and deployed.

### Application Components
- **Vote Service (Python)** – User-facing voting interface
- **Result Service (Node.js)** – Real-time results visualization
- **Worker Service (.NET)** – Background vote processing
- **Redis** – In-memory message broker
- **PostgreSQL** – Persistent relational database

### Infrastructure & Tooling
- **Amazon EKS** – Managed Kubernetes cluster
- **Docker** – Containerization
- **NGINX Ingress Controller** – HTTP routing
- **GitHub Actions** – CI/CD automation
- **Docker Hub** – Container image registry

### Data Flow
Vote → Redis → Worker → PostgreSQL → Result

yaml
Copiar código

---

## Objectives

### 1. Kubernetes Deployment
- Containerize all microservices
- Deploy each service using Kubernetes **Deployments** and **Services**
- Ensure inter-service communication via Kubernetes **internal DNS**
- Avoid hardcoded IPs or environment-specific dependencies

### 2. CI/CD Automation
- Automatically build Docker images on code changes
- Push images to a container registry
- Deploy and update Kubernetes resources on EKS
- Enable reproducible, zero-downtime deployments

---

## Repository Structure

.
├── K8s/
│ ├── redis/
│ ├── postgres/
│ ├── vote/
│ ├── result/
│ ├── worker/
│ └── ingress.yaml
├── docker/
│ ├── vote/
│ ├── result/
│ └── worker/
├── .github/
│ └── workflows/
│ └── ci-cd-pipeline.yml
└── README.md

yaml
Copiar código

---

## Kubernetes Design Considerations

- Each microservice runs in its own **Deployment**
- Services are exposed internally using **ClusterIP**
- Communication is performed via service DNS names, e.g.:
redis-service.default.svc.cluster.local

yaml
Copiar código
- This design ensures scalability, resilience, and portability across environments

---

## Ingress & Traffic Management

The application is exposed externally through an **NGINX Ingress Controller**, installed via Helm.

### Routing Rules
| Path     | Service |
|----------|--------|
| `/vote`  | Vote Service |
| `/result`| Result Service |

This enables clean URL-based routing without exposing individual services.

---

## CI/CD Pipeline (GitHub Actions)

The pipeline is triggered on pushes to the main branch and performs the following steps:

1. Build Docker images for each microservice
2. Push images to Docker Hub
3. Configure AWS credentials securely via GitHub Secrets
4. Authenticate to the EKS cluster
5. Apply Kubernetes manifests using `kubectl`

### AWS Credentials Configuration
```yaml
- name: Configure AWS Credentials
uses: aws-actions/configure-aws-credentials@v1
with:
  aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
  aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  aws-region: eu-west-3
This step allows the pipeline to securely interact with AWS services without exposing credentials in the repository.

EKS Authentication
yaml
Copiar código
- name: Install eksctl
  run: |
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin

- name: Update Kubeconfig
  run: eksctl utils write-kubeconfig --cluster <cluster-name> --region us-east-1
These steps ensure the CI runner can authenticate and communicate with the target EKS cluster.

Deployment to Kubernetes
yaml
Copiar código
- name: Apply Kubernetes Manifests
  run: kubectl apply -f K8s/
Applies all Kubernetes resources declaratively, enabling idempotent deployments.

Validation & Operations
Verify Workloads
bash
Copiar código
kubectl get deployments
kubectl get pods
Confirms that all services are running and healthy.

Verify Ingress
bash
Copiar código
kubectl get ingress
Retrieves the external IP or DNS assigned to the application.

Application Access
Once the ingress is available:

Vote Application

arduino
Copiar código
http://<INGRESS_IP>/vote
Results Application

arduino
Copiar código
http://<INGRESS_IP>/result
Votes submitted through the Vote service propagate through Redis, are processed by the Worker, persisted in PostgreSQL, and visualized in the Result service.

Secrets Management
Sensitive data is handled using Kubernetes Secrets, avoiding plaintext credentials in manifests.

Create Database Credentials
bash
Copiar código
kubectl create secret generic db-credentials \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=strongPassword
Inject Secrets into Deployments
yaml
Copiar código
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
This approach aligns with security best practices for containerized workloads.

Key Technologies
Amazon EKS

Kubernetes

Docker

GitHub Actions

NGINX Ingress Controller

Redis

PostgreSQL

Python, Node.js, .NET

Outcome
This project showcases practical expertise in:

Designing and operating Kubernetes-based microservices

Implementing production-grade CI/CD pipelines

Managing cloud infrastructure on AWS

Applying DevOps and cloud-native best practices

It serves as a strong portfolio example for Cloud Engineer, DevOps Engineer, or Platform Engineer roles.

