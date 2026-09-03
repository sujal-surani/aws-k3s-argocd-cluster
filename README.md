# AWS-K3s-ArgoCD-Cluster

A fully automated, declarative GitOps deployment pipeline provisioning an AWS-based Kubernetes cluster and managing application state via continuous synchronization.

## Architecture Overview

This repository acts as the single source of truth for both infrastructure and application configurations. It utilizes a pull-based GitOps methodology to ensure the live cloud environment perfectly matches the declared code.

* **Infrastructure as Code (IaC):** Terraform dynamically provisions AWS resources (EC2 `t3.small`, IAM Instance Profiles, and Security Groups).
* **Kubernetes Control Plane:** A lightweight K3s cluster bootstrapped on Ubuntu, managed remotely via AWS Systems Manager (SSM) without exposing SSH.
* **Continuous Deployment (GitOps):** ArgoCD actively monitors the `k8s/` directory in this repository, automatically deploying updates and preventing state drift.
* **Traffic Routing:** Traefik Ingress controller routes external internet traffic into the internal pod network.

## Repository Structure

```text
.
├── git.sh                  # Automation script for Git operations
├── k8s/                    # Kubernetes declarative manifests
│   ├── deployment.yaml     # Web application pod specifications 
│   ├── ingress.yaml        # Traefik routing rules for public access
│   └── service.yaml        # Internal networking for application pods
├── README.md               # Project documentation
└── terraform/              # Infrastructure as Code (AWS)
    ├── iam.tf              # SSM agent roles and permissions
    ├── main.tf             # EC2 instance provisioning
    ├── outputs.tf          # Dynamic public IP extraction
    ├── providers.tf        # AWS provider configuration
    └── security.tf         # Firewall rules (Ports 80, 6443)
```
*(Note: Terraform state files and local `.terraform/` directories are securely excluded via `.gitignore` to prevent secret leakage).*

## Deployment Lifecycle

### 1. Provision Infrastructure
Navigate to the `terraform/` directory to deploy the physical AWS resources.
```bash
cd terraform
terraform init
terraform apply
```

### 2. Bootstrap Kubernetes
Connect to the EC2 host securely via AWS SSM and install the K3s control plane, binding the API to the public IP.
```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--tls-san <AWS_PUBLIC_IP>" sh -
```
Extract the generated `/etc/rancher/k3s/k3s.yaml` to the local workstation's `~/.kube/config` to enable remote `kubectl` access.

### 3. Initialize GitOps
Install ArgoCD directly into the cluster from the local workstation.
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 4. Automated Synchronization
Once the ArgoCD application is configured to track the `k8s/` path of this repository, any git commits (e.g., scaling replicas in `deployment.yaml` or exposing services in `ingress.yaml`) are automatically detected and deployed without manual intervention.

---

## Contact

**Sujal Surani** - [https://www.linkedin.com/in/sujal-surani/]

## Maintainer

Created and maintained by **Sujal Surani**.