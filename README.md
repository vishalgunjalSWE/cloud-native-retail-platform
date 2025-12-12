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

