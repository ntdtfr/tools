# Gravitee

This repository contains Kubernetes manifests for deploying Gravitee API Management platform using Skaffold, Kustomize, and Sealed Secrets.

## Table of Contents

- [Gravitee](#gravitee)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Repository Structure](#repository-structure)
  - [Getting Started](#getting-started)
  - [Deployment](#deployment)
    - [Development Environment](#development-environment)
    - [Staging Environment](#staging-environment)
    - [Production Environment](#production-environment)
    - [Continuous Development](#continuous-development)
  - [Configuration](#configuration)
    - [Environment-specific Configuration](#environment-specific-configuration)
    - [Scaling](#scaling)
  - [Security Considerations](#security-considerations)
  - [Monitoring and Observability](#monitoring-and-observability)
  - [Backup and Disaster Recovery](#backup-and-disaster-recovery)
  - [Maintenance](#maintenance)
    - [Upgrades](#upgrades)
    - [Scaling](#scaling-1)
  - [Troubleshooting](#troubleshooting)
    - [Common Issues](#common-issues)

## Overview

This project provides a production-ready setup for deploying the Gravitee API Management platform on Kubernetes. The deployment includes:

- API Gateway
- Management API
- Management UI
- Access Management
- Alert Engine

The setup follows best practices for:
- **Security**: RBAC, network policies, pod security contexts, sealed secrets
- **Maintainability**: Kustomize for environment management, Skaffold for CI/CD integration
- **Resilience**: Probes, resource limits, pod disruption budgets, affinity rules
- **Scalability**: Horizontal pod autoscaling, proper resource requests/limits

## Prerequisites

- Kubernetes 1.22+
- [kubectl](https://kubernetes.io/docs/tasks/tools/) 1.22+
- [Skaffold](https://skaffold.dev/docs/install/) v2.0.0+
- [Kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/) v4.5.0+
- [kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal) v0.18.0+
- [Sealed Secrets Controller](https://github.com/bitnami-labs/sealed-secrets#controller) installed in your cluster

## Repository Structure

```
.
├── README.md
├── skaffold.yaml              # Skaffold configuration
├── base/                      # Base Kustomize configuration
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── api-gateway/           # API Gateway component
│   ├── management-api/        # Management API component
│   ├── management-ui/         # Management UI component
│   ├── access-management/     # Access Management component
│   ├── alert-engine/          # Alert Engine component
│   ├── mongodb/               # MongoDB database
│   ├── elasticsearch/         # Elasticsearch database
│   └── redis/                 # Redis cache
├── overlays/                  # Environment-specific configurations
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── sealed-secrets/
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── sealed-secrets/
│   └── production/
│       ├── kustomization.yaml
│       └── sealed-secrets/
└── manifests/                 # Generated manifests (gitignored)
```

## Getting Started

1. Clone this repository:
   ```bash
   git clone https://github.com/your-org/gravitee-k8s-deployment.git
   cd gravitee-k8s-deployment
   ```

2. Install Sealed Secrets controller if not already installed:
   ```bash
   helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
   helm install sealed-secrets -n kube-system sealed-secrets/sealed-secrets
   ```

3. Generate sealed secrets for your environment:
   ```bash
   # Example for creating a sealed secret
   kubectl create secret generic gravitee-mongodb-credentials \
     --from-literal=username=admin \
     --from-literal=password=yourpassword \
     --dry-run=client -o yaml | \
     kubeseal --controller-name=sealed-secrets \
     --controller-namespace=kube-system \
     --format yaml > overlays/dev/sealed-secrets/mongodb-credentials.yaml
   ```

## Deployment

### Development Environment

```bash
skaffold run -p dev
```

### Staging Environment

```bash
skaffold run -p staging
```

### Production Environment

```bash
skaffold run -p production
```

### Continuous Development

For local development with automatic redeployment on changes:

```bash
skaffold dev -p dev
```

## Configuration

### Environment-specific Configuration

Each environment (dev, staging, production) has its own configuration in the `overlays/` directory. Customize these files to match your environment's requirements.

### Scaling

Adjust resource requests/limits and replicas in the respective environment overlays.

## Security Considerations

This deployment follows these security best practices:

1. **RBAC**: Strict role-based access control for all components
2. **Sealed Secrets**: Sensitive data encrypted and safe to store in git
3. **Network Policies**: Controlled communication between pods
4. **Pod Security Context**: Non-root users, read-only file systems
5. **Security Context Constraints**: Restricted privileges
6. **TLS**: Encrypted communication between components
7. **Secret Rotation**: Procedures for regular credential rotation

## Monitoring and Observability

The deployment includes:

1. **Prometheus metrics**: Exported by all components
2. **Liveness/Readiness probes**: Health monitoring for all pods
3. **Logging**: Structured JSON logs sent to stdout/stderr
4. **Tracing**: OpenTelemetry integration
5. **Dashboards**: Grafana dashboard templates provided

## Backup and Disaster Recovery

1. **Database Backups**: Scheduled backups for MongoDB and Elasticsearch
2. **State Management**: Persistent volumes for stateful components
3. **Disaster Recovery**: Instructions for restoring from backups

## Maintenance

### Upgrades

1. Update the image versions in the appropriate environment overlay
2. Run Skaffold with the updated configuration

### Scaling

Adjust the replica count and resource limits in the environment overlays.

## Troubleshooting

### Common Issues

1. **Pod Startup Failures**: Check logs with `kubectl logs -n gravitee <pod-name>`
2. **Database Connection Issues**: Verify secrets and network policies
3. **Resource Constraints**: Check resource usage with `kubectl top pods -n gravitee`
