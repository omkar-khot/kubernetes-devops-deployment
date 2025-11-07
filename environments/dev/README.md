# Development Environment Configuration

## Overview
Development environment for local testing and feature development. Low resource requirements, rapid iteration enabled.

## Specifications

| Aspect | Configuration |
|--------|----------------|
| **Namespace** | `dev` |
| **Replicas** | 1 |
| **CPU Request** | 250m |
| **CPU Limit** | 500m |
| **Memory Request** | 256Mi |
| **Memory Limit** | 512Mi |
| **Storage** | 5Gi |
| **Backups** | None |
| **Monitoring** | Prometheus + Dynatrace (basic) |
| **Logging** | ELK Stack (7 day retention) |
| **Secrets** | Vault (dev credentials) |

## Deployment

### Using Kustomize
