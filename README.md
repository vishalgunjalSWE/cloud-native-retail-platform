# 🏪 Cloud-Native Retail Platform on AWS EKS

[![AWS](https://img.shields.io/badge/AWS-EKS%20%7C%20VPC%20%7C%20RDS-orange?logo=amazon-aws)](https://aws.amazon.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28-blue?logo=kubernetes)](https://kubernetes.io/)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-purple?logo=terraform)](https://www.terraform.io/)
[![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-red?logo=argo)](https://argo-cd.readthedocs.io/)
[![Go](https://img.shields.io/badge/Go-Microservices-00ADD8?logo=go)](https://golang.org/)

> **Production-grade microservices platform** built to demonstrate cloud-native architecture, event-driven patterns, GitOps automation, and comprehensive observability on AWS EKS.

![Platform Preview](https://github.com/vishalgunjalSWE/cloud-native-retail-platform/blob/49d2bd528f93b053212ce2b1bf512c8d2f31c96b/docs/images/banner.png)

---

## 📋 Table of Contents
- [Overview](#-overview)
- [Why I Built This](#-why-i-built-this)
- [Architecture](#-architecture)
- [Key Technical Features](#-key-technical-features)
- [Tech Stack](#-tech-stack)
- [What This Demonstrates](#-what-this-demonstrates)
- [Screenshots](#-screenshots)
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [What I Learned](#-what-i-learned)
- [Challenges Overcome](#-challenges-overcome)
- [Future Enhancements](#-future-enhancements)

---

## 🎯 Overview

**Personal learning project** showcasing production-grade microservices architecture on AWS EKS. Built a complete e-commerce platform with 8 independent services, event-driven communication, full GitOps automation, and enterprise-level observability.

This isn't a tutorial follow-along—every component was architected and implemented to understand real-world cloud-native patterns used by companies like Netflix, Uber, and Airbnb.

---

## 💡 Why I Built This

### **Learning Goals:**
- 🎯 Master **microservices architecture** and service decomposition patterns
- 🎯 Understand **event-driven communication** and async messaging (RabbitMQ)
- 🎯 Gain hands-on experience with **Kubernetes on AWS EKS** at scale
- 🎯 Learn **GitOps principles** with ArgoCD for declarative deployments
- 🎯 Build **production-grade observability** (Prometheus, Grafana, Jaeger)
- 🎯 Practice **Infrastructure as Code** with Terraform for reproducible environments
- 🎯 Implement **Kubernetes security** best practices (RBAC, Network Policies, IRSA)

### **What Makes This Different:**
- ✅ **8 independent microservices** (not a monolith split into containers)
- ✅ **Real event-driven patterns** (async messaging, pub/sub, dead-letter queues)
- ✅ **Production-grade Terraform** (modular, reusable, with remote state)
- ✅ **Complete observability** (metrics, logs, traces - all three pillars)
- ✅ **Security hardening** (RBAC, Network Policies, IRSA, Pod Security Standards)
- ✅ **GitOps workflow** (Git as single source of truth, automated sync)

---

## 🏗️ Architecture

<p align="center">
  <img src="https://github.com/vishalgunjalSWE/cloud-native-retail-platform/blob/67db042bf65bc77fe26963d9dd1588ebcd69b976/docs/images/EKS.gif" alt="System Architecture" width="800"/>
</p>

### **High-Level Architecture:**
```
User Request → AWS ALB/NLB → NGINX Ingress Controller
    ↓
API Gateway Service (Go)
    ↓
┌──────────────────────────────────────────────────┐
│  Microservices Layer (Event-Driven)              │
├──────────────────────────────────────────────────┤
│  Product Catalog (Go)  ←→  RabbitMQ  ←→  Orders │
│  Inventory (Go)        ←→  RabbitMQ  ←→  Payment│
│  User Management (Node) ←→  RabbitMQ  ←→  Notify│
└──────────────────────────────────────────────────┘
    ↓                           ↓
RDS PostgreSQL           ElastiCache Redis
(Multi-AZ)              (Caching Layer)
    ↓
Observability Stack
├─ Prometheus (Metrics)
├─ Grafana (Dashboards)
├─ Jaeger (Distributed Tracing)
└─ ELK Stack (Centralized Logs)
```

### **AWS Infrastructure:**
- **VPC:** Multi-AZ setup (3 availability zones)
- **EKS Cluster:** Managed Kubernetes with 3 worker nodes (t3.medium)
- **RDS PostgreSQL:** Multi-AZ deployment with automated backups
- **ElastiCache Redis:** Distributed caching layer
- **S3:** Static assets and Terraform state storage
- **Route53:** DNS management
- **ALB/NLB:** Load balancing for ingress traffic

---

## ✨ Key Technical Features

### **1. Microservices Architecture (8 Services)**

| Service | Language | Responsibility | Database |
|---------|----------|---------------|----------|
| **API Gateway** | Go (Gin) | Request routing, rate limiting, authentication | - |
| **Product Catalog** | Go | Product CRUD, search, filtering | PostgreSQL |
| **Inventory** | Go | Stock management, availability checks | PostgreSQL |
| **Order Processing** | Java (Spring Boot) | Order lifecycle, state management | PostgreSQL |
| **Payment Gateway** | Java (Spring Boot) | Payment processing, transaction handling | PostgreSQL |
| **User Management** | Node.js (Express) | User auth, profiles, preferences | PostgreSQL |
| **Notification** | Node.js (Express) | Email/SMS notifications | - |
| **Admin Dashboard** | React | Admin interface for management | - |

**Why this matters:** Each service is independently deployable, scalable, and maintainable. Demonstrates understanding of domain-driven design and service boundaries.

---

### **2. Event-Driven Communication (RabbitMQ)**

**Pattern Implementation:**
```
Order Created Event
    ↓
RabbitMQ Exchange (topic)
    ↓
├─ Inventory Queue → Update stock levels
├─ Payment Queue → Process payment
├─ Notification Queue → Send order confirmation email
└─ Analytics Queue → Track metrics
```

**Why event-driven?**
- **Loose coupling:** Services don't directly depend on each other
- **Resilience:** If Notification service is down, order processing continues
- **Scalability:** Can add consumers without changing producers
- **Async processing:** Long-running tasks don't block user requests

**Real implementation details:**
- Message durability (survives RabbitMQ restart)
- Dead-letter queues for failed messages
- Retry logic with exponential backoff
- Message acknowledgment patterns

---

### **3. Distributed Caching (Redis)**

**Caching Strategy:**
```
User Request → Check Redis Cache
    ↓
Cache HIT → Return cached data (sub-10ms response)
    ↓
Cache MISS → Query PostgreSQL → Update Cache → Return data
```

**What's cached:**
- Product catalog data (high read, low write)
- User session data (authentication tokens)
- Shopping cart state (temporary data)
- Frequently accessed queries (top products, categories)

**Cache patterns used:**
- **Cache-aside** (lazy loading)
- **Write-through** (cart updates)
- **TTL-based expiration** (product data: 5 min, sessions: 30 min)
- **LRU eviction policy** when memory limit reached

---

### **4. AWS Infrastructure (Terraform)**

**Modular Terraform Structure:**
```
terraform/
├── modules/
│   ├── vpc/          # VPC, subnets, NAT, route tables
│   ├── eks/          # EKS cluster, node groups, add-ons
│   ├── rds/          # RDS PostgreSQL, parameter groups
│   ├── elasticache/  # Redis cluster
│   └── s3/           # S3 buckets, lifecycle policies
├── environments/
│   ├── dev.tfvars
│   ├── staging.tfvars
│   └── prod.tfvars
└── main.tf
```

**Infrastructure Highlights:**
- **Multi-AZ VPC:** 3 public subnets, 3 private subnets, NAT gateways
- **EKS Cluster:** Managed control plane, custom launch templates, auto-scaling node groups
- **Remote State:** S3 backend with DynamoDB locking (prevents concurrent modifications)
- **Workspace Isolation:** Separate state files for dev/staging/prod
- **Security:** VPC flow logs, encrypted RDS, encrypted S3, security groups with least privilege

**Why Terraform?**
- Entire infrastructure is reproducible from code
- Can spin up identical environments in minutes
- Version control for infrastructure changes
- Disaster recovery (destroy + recreate from state)

---

### **5. Kubernetes Security Hardening**

**Security Layers Implemented:**

**RBAC (Role-Based Access Control):**
```yaml
# Each microservice has dedicated ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: product-catalog-sa
  namespace: retail

# Namespace-scoped Role (least privilege)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: product-catalog-role
rules:
- apiGroups: [""]
  resources: ["pods", "configmaps"]
  verbs: ["get", "list"]
```

**IAM Roles for Service Accounts (IRSA):**
- Pods access AWS services (S3, RDS, Secrets Manager) **without hardcoded credentials**
- Each service has IAM role with minimum required permissions
- No AWS keys stored in containers or environment variables

**Network Policies:**
```yaml
# Default deny all traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

# Explicit allow: Only Order service can call Payment service
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-order-to-payment
spec:
  podSelector:
    matchLabels:
      app: payment-service
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: order-service
```

**Pod Security Standards:**
- **Restricted profile** enforced (no privileged containers)
- Run as non-root user (UID 1000)
- Read-only root filesystem
- Drop all Linux capabilities except required ones
- Seccomp profiles for syscall filtering

---

### **6. GitOps with ArgoCD**

**How GitOps Works:**
```
Developer commits code → GitHub
    ↓
CI Pipeline builds Docker image
    ↓
Update image tag in Git manifest repo
    ↓
ArgoCD detects change (polls every 3 min)
    ↓
ArgoCD syncs to Kubernetes cluster
    ↓
Pods updated with new image (rolling update)
    ↓
Health checks verify deployment success
    ↓
If failed → Automated rollback to previous version
```

**Benefits of GitOps:**
- **Git as single source of truth** (audit trail of all changes)
- **Automated deployments** (no manual kubectl apply)
- **Self-healing** (ArgoCD corrects manual changes to cluster)
- **Disaster recovery** (rebuild cluster from Git)
- **Environment parity** (dev/staging/prod use same manifests with overlays)

**ArgoCD Features Used:**
- Auto-sync with self-healing
- Progressive rollout (10% → 50% → 100%)
- Health assessment (pods ready, services available)
- Slack notifications for sync events
- RBAC for team-based access

---

### **7. Comprehensive Observability**

**Three Pillars of Observability:**

**Metrics (Prometheus + Grafana):**
```
Prometheus scrapes metrics from:
├─ Microservices (/metrics endpoints - RED metrics)
├─ node-exporter (CPU, memory, disk, network per node)
├─ kube-state-metrics (pod status, deployments, replicas)
└─ RabbitMQ exporter (queue depth, message rates)

Grafana Dashboards:
├─ Cluster Overview (node health, resource utilization)
├─ Namespace Metrics (CPU/memory per namespace)
├─ Application RED (Request rate, Error rate, Duration)
├─ RabbitMQ (queue depth, consumer lag, throughput)
└─ SLO Tracking (99.9% availability target, p95 latency)
```

**Logs (ELK Stack - Optional):**
```
Filebeat collects logs from pods
    ↓
Logstash parses + enriches logs (add metadata)
    ↓
Elasticsearch stores + indexes logs
    ↓
Kibana for searching + visualization
```

**Traces (Jaeger):**
```
Distributed tracing across microservices:
User Request → API Gateway → Product Catalog → Inventory
    ↓
Jaeger shows:
- Total request time: 245ms
- API Gateway: 5ms
- Product Catalog: 120ms (DB query slow!)
- Inventory: 120ms
```

**Why all three?**
- **Metrics:** "What's happening?" (high CPU, many errors)
- **Logs:** "Why is it happening?" (error messages, stack traces)
- **Traces:** "Where is the bottleneck?" (which service is slow)

---

## 🛠️ Tech Stack

### **Languages & Frameworks**
- **Go (Golang):** Product Catalog, Inventory, API Gateway (concurrent request handling)
- **Java (Spring Boot):** Order Processing, Payment Gateway (transaction management)
- **Node.js (Express):** User Management, Notifications (async I/O operations)
- **React:** Admin Dashboard frontend

### **Infrastructure & Cloud**
- **AWS:** EKS, VPC, RDS (PostgreSQL), ElastiCache (Redis), S3, Route53, ALB/NLB, IAM
- **Terraform:** Infrastructure as Code (modular, reusable modules)
- **Kubernetes (EKS):** Container orchestration (1.28)
- **Docker:** Multi-stage builds for optimized images

### **Messaging & Caching**
- **RabbitMQ:** Event-driven messaging (pub/sub, topic exchanges, DLQ)
- **Redis:** Distributed caching, session storage

### **GitOps & CI/CD**
- **ArgoCD:** Declarative GitOps for Kubernetes deployments
- **GitHub Actions:** CI pipeline (build, test, security scan)
- **Helm:** Kubernetes package management

### **Observability**
- **Prometheus:** Metric collection and storage
- **Grafana:** Dashboard visualization, alerting
- **Jaeger:** Distributed tracing
- **Alertmanager:** Alert routing and notifications

### **Security**
- **Trivy:** Container image vulnerability scanning
- **RBAC:** Kubernetes role-based access control
- **Network Policies:** Pod-to-pod traffic restrictions
- **IRSA:** IAM roles for service accounts (no hardcoded credentials)

---

## 📚 What This Demonstrates

### **Cloud-Native Architecture Skills:**
- ✅ Microservices decomposition and domain-driven design
- ✅ Event-driven architecture with message queues
- ✅ Distributed caching strategies (cache-aside, write-through)
- ✅ API Gateway pattern for centralized routing
- ✅ Database per service pattern (data isolation)

### **Kubernetes Expertise:**
- ✅ EKS cluster provisioning and management
- ✅ Deployments, Services, Ingress, ConfigMaps, Secrets
- ✅ HPA (Horizontal Pod Autoscaling) based on CPU/memory
- ✅ RBAC for fine-grained access control
- ✅ Network Policies for pod-to-pod communication
- ✅ Pod Security Standards (restricted profile)
- ✅ Liveness/Readiness probes for health checking
- ✅ Resource limits and requests for QoS

### **Infrastructure as Code:**
- ✅ Modular Terraform for AWS infrastructure
- ✅ Remote state management (S3 + DynamoDB locking)
- ✅ Workspace-based environment separation
- ✅ VPC design (public/private subnets, NAT, routing)
- ✅ Security groups and IAM policies

### **GitOps & Automation:**
- ✅ ArgoCD for declarative deployments
- ✅ Git as single source of truth
- ✅ Automated sync and self-healing
- ✅ Progressive rollouts with health checks

### **Observability & SRE:**
- ✅ Prometheus metric collection (custom exporters)
- ✅ Grafana dashboard design (RED metrics, SLO tracking)
- ✅ Distributed tracing with Jaeger
- ✅ Alert configuration with Alertmanager
- ✅ Understanding of SLIs, SLOs, error budgets

### **Security Best Practices:**
- ✅ Least-privilege IAM roles (IRSA)
- ✅ No hardcoded credentials (Kubernetes secrets, AWS Secrets Manager)
- ✅ Network segmentation (Network Policies)
- ✅ Container security (non-root users, read-only filesystem)
- ✅ RBAC for access control

---

<!-- ## 📸 Screenshots

<details>
<summary><b>🖼️ Click to view platform screenshots</b></summary>

### **Architecture Diagram**
![Architecture](./docs/architecture-diagram.png)
*Complete system architecture showing all microservices, data flow, and infrastructure components*

### **Kubernetes Cluster (kubectl)**
![Kubectl](./screenshots/kubectl-pods.png)
*All microservices running in Kubernetes cluster with ready status*

### **ArgoCD GitOps Dashboard**
![ArgoCD](./screenshots/argocd-sync.png)
*Application synced and healthy, showing GitOps automation in action*

### **Grafana: Cluster Metrics**
![Grafana Cluster](./screenshots/grafana-cluster.png)
*Node-level resource utilization (CPU, memory, network, disk)*

### **Grafana: Application Metrics (RED)**
![Grafana RED](./screenshots/grafana-red-metrics.png)
*Request rate, error rate, and duration metrics per microservice*

### **Grafana: Namespace Overview**
![Grafana Namespace](./screenshots/grafana-namespace.png)
*Resource consumption per namespace for capacity planning*

### **Prometheus Targets**
![Prometheus](./screenshots/prometheus-targets.png)
*All monitoring targets up and successfully scraped*

### **Jaeger Distributed Tracing**
![Jaeger](./screenshots/jaeger-trace.png)
*Request trace showing latency breakdown across microservices*

</details>

--- -->

## 🚀 Quick Start

### **Prerequisites**
- AWS Account (free tier eligible)
- `kubectl` v1.28+
- `terraform` v1.6+
- `helm` v3.12+
- `aws-cli` configured with credentials

### **1. Clone Repository**
```bash
git clone https://github.com/VishalGunjalswe/cloud-native-retail-platform.git
cd cloud-native-retail-platform
```

### **2. Provision Infrastructure with Terraform**
```bash
cd terraform/

# Initialize Terraform
terraform init

# Create dev workspace
terraform workspace new dev

# Review infrastructure plan
terraform plan -var-file=environments/dev.tfvars

# Apply infrastructure (creates VPC, EKS, RDS, Redis)
terraform apply -var-file=environments/dev.tfvars -auto-approve

# Update kubeconfig
aws eks update-kubeconfig --region us-east-1 --name retail-platform-dev
```

### **3. Deploy Observability Stack**
```bash
# Install Prometheus + Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Get Grafana admin password
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port-forward to access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# Open http://localhost:3000 (admin / <password>)
```

### **4. Install ArgoCD**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get ArgoCD admin password
argocd admin initial-password -n argocd

# Port-forward to access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open https://localhost:8080 (admin / <password>)
```

### **5. Deploy Application via ArgoCD**
```bash
# Create ArgoCD application
argocd app create retail-platform \
  --repo https://github.com/VishalGunjalswe/cloud-native-retail-platform \
  --path k8s/overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --self-heal

# Watch deployment progress
argocd app get retail-platform --watch
```

### **6. Access Application**
```bash
# Get LoadBalancer URL
kubectl get svc -n ingress-nginx

# Application will be available at:
# http://<LoadBalancer-URL>
```

---

## 📁 Project Structure

```
cloud-native-retail-platform/
├── terraform/                    # Infrastructure as Code
│   ├── modules/
│   │   ├── vpc/                 # VPC, subnets, NAT, route tables
│   │   ├── eks/                 # EKS cluster, node groups
│   │   ├── rds/                 # RDS PostgreSQL
│   │   ├── elasticache/         # Redis cluster
│   │   └── s3/                  # S3 buckets
│   ├── environments/
│   │   ├── dev.tfvars
│   │   ├── staging.tfvars
│   │   └── prod.tfvars
│   └── main.tf
│
├── microservices/               # Service source code
│   ├── api-gateway/             # Go - Request routing
│   ├── product-catalog/         # Go - Product CRUD
│   ├── inventory/               # Go - Stock management
│   ├── order-processing/        # Java - Order lifecycle
│   ├── payment-gateway/         # Java - Payments
│   ├── user-management/         # Node.js - User auth
│   ├── notification/            # Node.js - Notifications
│   └── admin-dashboard/         # React - Admin UI
│
├── k8s/                         # Kubernetes manifests
│   ├── base/                    # Base Kustomize configs
│   │   ├── deployments/
│   │   ├── services/
│   │   ├── configmaps/
│   │   └── secrets/
│   └── overlays/                # Environment overlays
│       ├── dev/
│       ├── staging/
│       └── prod/
│
├── helm-charts/                 # Helm charts
│   ├── retail-platform/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   └── monitoring/
│
├── observability/               # Monitoring configs
│   ├── prometheus/
│   │   └── service-monitors.yaml
│   ├── grafana/
│   │   └── dashboards/
│   │       ├── cluster-overview.json
│   │       ├── namespace-metrics.json
│   │       └── red-metrics.json
│   └── jaeger/
│       └── jaeger-deployment.yaml
│
├── docs/                        # Documentation
│   ├── ARCHITECTURE.md          # Detailed architecture
│   ├── DEPLOYMENT.md            # Deployment guide
│   ├── SECURITY.md              # Security implementation
│   └── MONITORING.md            # Observability setup
│
├── screenshots/                 # Platform screenshots
├── scripts/                     # Automation scripts
└── README.md
```

---

## 🎓 What I Learned

### **1. Microservices Architecture**
- **Service decomposition is hard:** Deciding service boundaries requires understanding business domains
- **Distributed transactions are complex:** Need sagas, compensation, idempotency
- **Network calls fail:** Retry logic, circuit breakers, timeouts are essential
- **Data consistency is tricky:** Eventual consistency vs strong consistency trade-offs

### **2. Event-Driven Architecture**
- **Async is powerful but complex:** Debugging message flows is harder than sync APIs
- **Message ordering matters:** FIFO queues needed for certain use cases
- **Dead-letter queues are critical:** Failed messages need separate handling
- **Idempotency is non-negotiable:** Messages can be delivered multiple times

### **3. Kubernetes at Scale**
- **Resource limits prevent noisy neighbors:** CPU/memory requests/limits are crucial
- **Liveness vs Readiness probes:** Know when to use each
- **HPA based on custom metrics:** CPU alone isn't enough for scaling decisions
- **Network Policies are tedious but necessary:** Default deny-all is safest starting point

### **4. Infrastructure as Code**
- **Modular Terraform reduces duplication:** Modules make infrastructure reusable
- **Remote state is essential for teams:** S3 + DynamoDB locking prevents conflicts
- **Drift happens:** Manual changes to infrastructure cause unexpected issues
- **Terraform plan before apply:** Always review changes before execution

### **5. GitOps in Practice**
- **Git as single source of truth works:** Confidence in deployments increased significantly
- **Self-healing is amazing:** ArgoCD auto-corrects manual kubectl changes
- **Sync waves matter:** Order of resource creation/updates is important
- **Helm hooks for migrations:** Database schema updates need careful orchestration

### **6. Observability**
- **Metrics alone aren't enough:** Need logs and traces for complete picture
- **Cardinality matters:** Too many unique label combinations overwhelm Prometheus
- **Dashboards need context:** Raw metrics without business context are useless
- **Alerting is an art:** Too many alerts = alert fatigue, too few = missed incidents

### **7. Security is Hard**
- **Principle of least privilege:** Give minimum permissions required, then expand if needed
- **Defense in depth:** Multiple security layers catch what others miss
- **Secrets management is critical:** Never hardcode credentials, use IRSA/Secrets Manager
- **Network segmentation helps:** Network Policies limit blast radius of compromises

---

## 🔥 Challenges Overcome

### **1. EKS Networking**
**Challenge:** Understanding VPC design, subnets (public/private), NAT gateways, security groups, and how pods get IPs.

**Solution:** Spent time learning AWS VPC networking, drew architecture diagrams, tested connectivity between components. Learned about AWS CNI plugin for pod networking.

### **2. RabbitMQ Message Loss**
**Challenge:** Messages were being lost when RabbitMQ pod restarted.

**Solution:** Configured message durability, persistent queues, and publisher confirms. Added PersistentVolume for RabbitMQ data.

### **3. Prometheus High Cardinality**
**Challenge:** Prometheus crashing due to too many unique metric label combinations.

**Solution:** Reduced label cardinality by removing high-variance labels (user IDs, request IDs). Used relabeling to drop unnecessary labels.

### **4. Inter-Service Authentication**
**Challenge:** How to securely authenticate service-to-service calls.

**Solution:** Implemented mutual TLS (mTLS) with service mesh concepts. Each service validates JWT tokens issued by auth service.

### **5. Database Connection Pooling**
**Challenge:** Services creating too many database connections, exhausting RDS limits.

**Solution:** Configured connection pooling in application code (HikariCP for Java, pgx for Go). Set max connections based on RDS instance size.

### **6. ArgoCD Out-of-Sync Issues**
**Challenge:** ArgoCD showing OutOfSync status even though cluster state matched Git.

**Solution:** Learned about ignoreDifferences for fields that change at runtime (replicas with HPA). Used sync options for resource pruning.

---

## 🔮 Future Enhancements

### **Short-Term (Next 1-2 months)**
- [ ] **Service Mesh (Istio):** Traffic management, circuit breakers, mTLS between services
- [ ] **Chaos Engineering:** Chaos Mesh for resilience testing (kill pods, inject network latency)
- [ ] **Canary Deployments:** Flagger for progressive rollouts (10% → 50% → 100%)
- [ ] **Cost Optimization:** AWS cost dashboards, right-sizing recommendations, spot instances

### **Medium-Term (3-6 months)**
- [ ] **Multi-Region Deployment:** Active-active across us-east-1 and us-west-2
- [ ] **CI/CD Pipeline:** Automated testing, security scanning, image building
- [ ] **GraphQL Federation:** Unified API gateway using GraphQL
- [ ] **Event Sourcing:** CQRS pattern for order and inventory services

### **Long-Term (6-12 months)**
- [ ] **Serverless Components:** AWS Lambda for certain services (notifications, webhooks)
- [ ] **ML-Based Recommendations:** Product recommendation engine using TensorFlow
- [ ] **Edge Deployment:** Deploy to AWS Wavelength for low latency
- [ ] **FinOps Automation:** Auto-scaling based on cost, not just CPU/memory

---

## 🤝 Contributing

This is a personal learning project, but feedback and suggestions are welcome!

1. Fork the repository
2. Create feature branch (`git checkout -b feature/improvement`)
3. Commit changes (`git commit -m 'Add improvement'`)
4. Push to branch (`git push origin feature/improvement`)
5. Open Pull Request

---

## 📝 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file.

---

## 📞 Contact & Further Information

This project represents comprehensive application of **DevSecOps, GitOps, and SRE principles** to deliver tangible business outcomes. It demonstrates my ability to architect, build, and operate modern cloud-native systems.

For further discussion about this project or potential opportunities, please connect:

<p align="left"> 
  <a href="https://www.linkedin.com/in/vishalgunjal1/" target="_blank">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn Badge"/>
  </a> 
  <a href="https://vishalgunjal-cv.vercel.app/" target="_blank">
    <img src="https://img.shields.io/badge/Portfolio-D14836?style=for-the-badge&logo=google-chrome&logoColor=white" alt="Portfolio Badge"/>
  </a>
  <a href="https://github.com/vishalgunjalswe" target="_blank">
    <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub Badge"/>
  </a>
  <a href="https://medium.com/@vishalgunjal0287" target="_blank">
    <img src="https://img.shields.io/badge/Medium-000000?style=for-the-badge&logo=medium&logoColor=white" alt="Medium Badge"/>
  </a>
</p>

---

**⭐ If you found this project useful, please star the repository!**

It helps others discover this work and motivates me to build more production-grade projects.

---

## 📊 Repository Stats

![GitHub last commit](https://img.shields.io/github/last-commit/vishalgunjalSWE/wanderlust-devsecops-gitops)
![GitHub repo size](https://img.shields.io/github/repo-size/vishalgunjalSWE/wanderlust-devsecops-gitops)
![GitHub language count](https://img.shields.io/github/languages/count/vishalgunjalSWE/wanderlust-devsecops-gitops)

**Built with ❤️ and ☕ in Pune, India**

---

**Engineering In Public | Happy Learning! :-)**