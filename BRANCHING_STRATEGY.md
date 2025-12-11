# Branching Strategy & GitOps Workflow

## 🏗️ Branch Architecture

This repository implements a **dual-branch GitOps strategy** designed for different deployment scenarios with clear separation between simple public deployments and production-ready automated workflows.

```mermaid
graph LR
    A[Developer] --> B[Code Changes]
    B --> C{Target Branch?}
    C -->|Simple Deployment| D[main branch]
    C -->|Production| E[gitops branch]
    
    D --> F[Manual Deployment]
    D --> G[Public ECR Images]
    D --> H[Umbrella Chart]
    
    E --> I[GitHub Actions]
    E --> J[Private ECR Images]
    E --> K[Individual Apps]
    
    I --> L[Build & Push]
    L --> M[Update Helm Charts]
    M --> N[ArgoCD Sync]
```

## 🌐 Public Application (Main Branch)

### **Purpose**
Simple deployment with public container images for demos, learning, and quick testing.

### **Characteristics**
```yaml
✅ Branch: main
✅ Images: Public ECR (stable versions)
✅ Deployment: Manual Helm chart management
✅ ArgoCD: Umbrella chart (retail-store-app)
✅ Workflows: None (no .github/workflows/)
✅ Updates: Manual only
✅ Target: Demos, learning, simple deployments
```