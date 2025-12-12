# Cloud-Native Retail Platform

<div align="center">

![Banner](./docs/images/banner.png)

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/vishalgunjal/cloud-native-retail-platform)
[![Terraform](https://img.shields.io/badge/terraform-v1.9+-purple)](https://www.terraform.io/)
[![Kubernetes](https://img.shields.io/badge/kubernetes-EKS-blue)](https://aws.amazon.com/eks/)
[![GitOps](https://img.shields.io/badge/GitOps-ArgoCD-orange)](https://argoproj.github.io/argo-cd/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

</div>

---

## 🚀 Overview

**Cloud-Native Retail Platform** is a production-ready, distributed e-commerce architecture designed to demonstrate **Site Reliability Engineering (SRE)** principles at scale.

Unlike traditional monolithic applications, this platform utilizes a **polyglot microservices architecture** (Go, Java, Node.js) orchestrated on **AWS EKS**. It implements strict **GitOps** workflows using ArgoCD for zero-touch deployments, **Terraform** for immutable infrastructure, and **RabbitMQ** for event-driven resilience.

### Key Engineering Goals
*   **Decoupling:** Asynchronous communication via message queues to prevent cascading failures.
*   **Immutability:** Infrastructure provisioning solely via code (IaC) using Terraform.
*   **Observability:** Comprehensive monitoring of pod health and service latency.
*   **GitOps:** Single source of truth for both application and infrastructure state.

---

## 🏗 Architecture

The system is composed of loose-coupled microservices communicating via REST and AMQP (RabbitMQ).

### Application Microservices

The retail store consists of 5 microservices working together:

| Service | Language | Responsibility | Port |
| :--- | :--- | :--- | :--- |
| **UI** | Java (Spring Boot) | Web interface (Frontend) | 8080 |
| **Catalog** | **Go (Golang)** | Product catalog API (High Read Traffic) | 8081 |
| **Cart** | Java (Spring Boot) | Shopping cart API (Redis Cache) | 8082 |
| **Orders** | Java (Spring Boot) | Order management API (Transactional) | 8083 |
| **Checkout** | Node.js (NestJS) | Orchestration & Async Processing | 8084 |

![Application Architecture](./docs/images/architecture.png)

### Infrastructure Architecture

The platform runs on a custom VPC with public/private subnets, utilizing **EKS Auto Mode** for compute management.

![EKS Infrastructure](./docs/images/EKS.gif)

---

## 🔄 GitOps Workflow

Utilize a "Pull-Based" GitOps deployment strategy. Application state is reconciled automatically by ArgoCD.

```mermaid
graph LR
    A[Code Push] --> B[GitHub Actions]
    B --> C[Build Images]
    C --> D[Push to ECR]
    D --> E[Update Helm Charts]
    E --> F[Commit Changes]
    F --> G[ArgoCD Sync]
    G --> H[Deploy to EKS]
````

## 🛠 Tech Stack

### ☁️ Infrastructure & Cloud

* **Cloud Provider:** AWS (Region: `us-west-2`)
* **Orchestrator:** Amazon EKS (Elastic Kubernetes Service) - *Auto Mode*
* **IaC:** Terraform (Modular structure for VPC, Security Groups, EKS)
* **Container Registry:** AWS ECR

### 🚀 DevOps & CI/CD

* **Version Control:** Git
* **CI Pipelines:** GitHub Actions (Linting, Docker Build, Push to ECR)
* **CD Controller:** ArgoCD (GitOps implementation)
* **Package Manager:** Helm Charts

### 🛡️ Observability & Security

* **Ingress:** Nginx Ingress Controller
* **Security:** AWS IAM Roles for Service Accounts (IRSA), Network Policies
* **Secrets:** Kubernetes Secrets (Opaque)

---

## ⚡ Getting Started

### Prerequisites

* AWS CLI configured with Admin permissions
* Terraform v1.5+
* Kubectl & Helm
* Docker Desktop

### Phase 1: Provision Infrastructure

We treat infrastructure as ephemeral. The entire VPC and Cluster are built from code.

```bash
cd terraform
terraform init
terraform plan
terraform apply --auto-approve
```

### Phase 2: Bootstrap Cluster

Update your local kubeconfig to interact with the new cluster.

```bash
aws eks update-kubeconfig --region us-west-2 --name cloud-native-retail
kubectl get nodes
```

### Phase 3: Deploy ArgoCD

Deploy the GitOps controller to manage the application state.

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## 🧠 Engineering Decisions & Trade-offs

### Why RabbitMQ?

Instead of synchronous HTTP calls between *Checkout* and *Order* services, I implemented RabbitMQ.

* **Benefit:** If the Order database is under high load, the Checkout service can still accept requests. Messages queue up and are processed when resources are available.
* **Trade-off:** Introduces "Eventual Consistency" complexity, which handled via idempotency keys.

### Why Go for Catalog?

The Catalog service receives the highest read-traffic (browsing).

* **Decision:** Golang was chosen for its low memory footprint and high concurrency handling compared to the Java services.

---

## 🤝 Future Roadmap

* [ ] **Cost Optimization:** Implement Karpenter for Just-in-Time node scaling.
* [ ] **Chaos Engineering:** Integrate Chaos Mesh to simulate pod failures in production.
* [ ] **Service Mesh:** Migrate to Istio for mTLS and better traffic splitting.

---

