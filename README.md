# AWS ECR + Docker + Minimal HTTP Demo

A lightweight web application running inside a BusyBox container on AWS EC2 and managed via Amazon Elastic Container Registry (ECR).

## 🚀 Overview

This repository contains a minimal HTTP web application served by BusyBox `httpd`. It demonstrates an end-to-end containerized deployment workflow on AWS, including CI/CD automation via GitHub Actions to build, push to Amazon ECR, and deploy to an AWS EC2 instance.

## 🏗️ Architecture Flow

```
[ Developer Push ] ──> [ GitHub Actions CI/CD ]
                              │
                              ├── (SSH into EC2)
                              │
                              ▼
                      [ AWS EC2 Instance ]
                       ├── Builds Docker Image
                       ├── Authenticates with Amazon ECR
                       ├── Pushes Image to ECR
                       └── Runs Container (Port 80)
```

1. **Infrastructure**: Provisioned AWS EC2 instance and Amazon ECR repository.
2. **Container Build**: Built using `busybox:1.36` serving static web content via `httpd`.
3. **Registry**: Pushed to Amazon Elastic Container Registry (ECR).
4. **Deployment**: Deployed and run on AWS EC2 listening on port 80.

---

## 📁 Repository Structure

```
.
├── Dockerfile                  # Lightweight Dockerfile based on busybox:1.36
├── index.html                  # Styled landing page with architecture details
└── .github/
    └── workflows/
        └── deploy.yml          # GitHub Actions deployment pipeline
```

---

## 🛠️ Local Development & Testing

### 1. Build the Docker Image

```bash
docker build -t aws-ecr-docker-minimal-http .
```

### 2. Run Container Locally

```bash
docker run -d -p 8080:80 --name my-app aws-ecr-docker-minimal-http
```

Access the application at `http://localhost:8080`.

### 3. Stop and Remove Local Container

```bash
docker stop my-app && docker rm my-app
```

---

## ⚙️ GitHub Actions CI/CD Deployment

The repository includes an automated deployment workflow defined in [.github/workflows/deploy.yml](file://.github/workflows/deploy.yml).

### Required GitHub Secrets

To enable automated deployments to EC2, configure the following secrets in your repository settings (**Settings > Secrets and variables > Actions**):

| Secret | Description |
| :--- | :--- |
| `EC2_HOST` | Public IP or DNS hostname of your AWS EC2 instance |
| `EC2_USERNAME` | SSH username (e.g., `ec2-user` or `ubuntu`) |
| `EC2_SSH_KEY` | Private SSH Key with access to the EC2 instance |
| `AWS_ACCOUNT_ID` | Your 12-digit AWS Account ID |

### Deployment Process

When changes are pushed to the `main` branch (or triggered manually via `workflow_dispatch`), the GitHub Action:
1. Connects to the EC2 instance via SSH (`appleboy/ssh-action`).
2. Clones or pulls the latest repository code on the host.
3. Builds the Docker image and authenticates to Amazon ECR.
4. Pushes the tagged image (`:latest`) to Amazon ECR (`eu-west-1`).
5. Restarts the container service (`my-app`) on port 80.
