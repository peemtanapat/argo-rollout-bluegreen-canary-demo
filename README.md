# Argo Rollouts Blue-Green & Canary Demo

A comprehensive demonstration of Argo Rollouts deployment strategies using Kubernetes.

## Overview

This repository contains practical examples of:
- **Blue-Green Deployments** with two different approaches
- **Canary Deployments** (work in progress)

## Project Structure

```
├── blue-green-rollout-with-nginx/     # Production-like setup with NGINX Ingress
├── blue-green-rollout-with-portforward/   # Simple setup with port-forwarding
└── canary-rollout/                    # Canary deployment examples
```

## Quick Start

### Prerequisites
- Kubernetes cluster (Docker Desktop recommended for local development)
- `kubectl` configured
- Argo Rollouts installed

### Installation
```bash
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install kubectl plugin (optional)
brew install argoproj/tap/kubectl-argo-rollouts
```

## Deployment Examples

### 1. Blue-Green with Port Forward
Simple approach for learning and testing:
```bash
cd blue-green-rollout-with-portforward/
kubectl apply -f blue-green-rollout.yaml
```

### 2. Blue-Green with NGINX Ingress
Production-like setup with stable domain names:
```bash
cd blue-green-rollout-with-nginx/
kubectl apply -f blue-green-rollout.yaml
kubectl apply -f api-ingress.yaml
```

### 3. Canary Deployment
Progressive traffic shifting:
```bash
cd canary-rollout/
kubectl apply -f canary-rollout.yaml
```

## Key Features Demonstrated

- ✅ Automated rollout strategies
- ✅ Manual promotion controls  
- ✅ Service mesh integration
- ✅ Zero-downtime deployments
- ✅ Rollback capabilities

## Learn More

Each subdirectory contains detailed README files with step-by-step instructions and explanations.

---

**Note**: This is a demo project for learning Argo Rollouts deployment strategies.