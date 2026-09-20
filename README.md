☁️ CivicLens — AWS Deployment

Deployment configuration for CivicLens: a single EC2 instance running Amazon Linux 2023, fronted by Nginx, with Session Manager for admin access and no inbound SSH.

![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?logo=amazon-aws&logoColor=white)
![IaC](https://img.shields.io/badge/IaC-CloudFormation-8A63D2)
![Access](https://img.shields.io/badge/access-SSM%20Session%20Manager-2E9CF0)
![License](https://img.shields.io/badge/license-MIT-2E9CF0)

---

## 📖 Table of Contents

- [🎯 Objective](#-objective)
- [📌 Architecture](#-architecture)
- [🧱 CloudFormation](#-cloudformation)
- [🗂️ Server Layout](#️-server-layout)
- [⚙️ Install Nginx](#️-install-nginx)
- [⚙️ Install the FastAPI Service](#️-install-the-fastapi-service)
- [🔒 Security](#-security)
- [👨‍💻 Author](#-author)

---

## 🎯 Objective

This directory contains everything needed to deploy CivicLens to AWS: a CloudFormation template for the compute and networking layer, and the Nginx/systemd configuration that runs on the instance itself.

---

## 📌 Architecture

```
                 Internet                    Admin · Session Manager
                    │                                   │
                    ▼                                   ▼
┌───────────────────────────────────────────────────────────────────┐
│  VPC (existing, public subnet)                                     │
│                                                                     │
│   ┌───────────────────────────────────────────────────────────┐   │
│   │  EC2 instance — Amazon Linux 2023                          │   │
│   │                                                             │   │
│   │   ┌─────────────────────┐        ┌─────────────────────┐   │   │
│   │   │  Nginx              │  /api/ │  FastAPI             │   │   │
│   │   │  Reverse proxy      │──────▶ │  Backend :8000        │   │   │
│   │   └─────────────────────┘        └─────────────────────┘   │   │
│   │                                                             │   │
│   └───────────────────────────────────────────────────────────┘   │
│                                                                     │
│   Security group: HTTP/80 + HTTPS/443 only · no inbound SSH        │
└───────────────────────────────────────────────────────────────────┘
```

| Service | Role |
|---|---|
| **Amazon EC2** (Amazon Linux 2023) | Hosts the React build and the FastAPI backend on a single instance. |
| **Nginx** | Serves `frontend/dist` and reverse-proxies `/api/` to FastAPI on `127.0.0.1:8000`. |
| **AWS Systems Manager (Session Manager)** | Administrative access to the instance without an open inbound SSH port. |
| **IAM Instance Profile** | Uses `AmazonSSMManagedInstanceCore` so Session Manager can connect. |
| **EC2 Security Group** | Publicly exposes only HTTP/80 and HTTPS/443. |
| **AWS CloudFormation** | Creates the EC2 instance, security group, IAM role, and instance profile in an existing VPC/subnet. |

---

## 🧱 CloudFormation

`civiclens-ec2-existing-vpc.yaml` expects an existing VPC, public subnet, and EC2 key pair.

The template deliberately does not contain AWS access keys, GitHub tokens, private keys, or other secrets.

---

## 🗂️ Server Layout

The deployment files assume:

| Path | Purpose |
|---|---|
| `/opt/CivicLens` | Application root |
| `/opt/CivicLens/backend` | Backend |
| `/opt/CivicLens/.venv` | Python virtual environment |
| `/opt/CivicLens/frontend/dist` | Frontend build |

---

## ⚙️ Install Nginx

On Amazon Linux 2023:

```bash
sudo dnf install -y nginx
sudo cp deployment/nginx/civiclens.conf /etc/nginx/conf.d/civiclens.conf
sudo nginx -t
sudo systemctl enable --now nginx
```

---

## ⚙️ Install the FastAPI Service

```bash
sudo cp deployment/systemd/civiclens.service /etc/systemd/system/civiclens.service
sudo systemctl daemon-reload
sudo systemctl enable --now civiclens
sudo systemctl status civiclens
```

---

## 🔒 Security

Do not commit:

- `.env` files containing secrets
- AWS access keys
- GitHub personal access tokens
- EC2 `.pem` private keys
- passwords or API keys

Use environment variables or AWS-managed identity/permissions for secrets.

---

## 👨‍💻 Author

**Arghya Roy**
🎓 B.Tech, Information Technology · ☁️ Cloud & DevOps Enthusiast

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/arghyaroy331)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/arghyaroy1/)

⭐ If you found this deployment setup useful, consider giving the project a star!
