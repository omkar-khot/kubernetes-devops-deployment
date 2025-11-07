# Multi-Environment Kubernetes Deployment with Helm

## Overview

This repository now supports **three separate Kubernetes environments**:

- **Development** - For developers testing new features
- **Staging** - Pre-production testing environment
- **Production** - Live production environment

## Directory Structure

The repository is organized by environment:

```
kubernetes-devops-deployment/
├── README.md                          # Main README
├── ENVIRONMENTS.md                    # This file
├── helm/
│   └── devops-app/
│       ├── Chart.yaml                 # Helm chart metadata
│       ├── values.yaml                # Default values
│       ├── values-dev.yaml            # Dev environment values
│       ├── values-staging.yaml        # Staging environment values
│       ├── values-prod.yaml           # Production environment values
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── ingress.yaml
│           ├── hpa.yaml
│           ├── configmap.yaml
│           ├── secret.yaml
│           └── _helpers.tpl
├── environments/
│   ├── dev/
│   │   └── README.md
│   ├── staging/
│   │   └── README.md
│   └── prod/
│       └── README.md
├── base/
│   ├── 01-namespaces/
│   ├── 02-secrets/
│   ├── 03-configmaps/
│   ├── 04-postgres/
│   ├── 05-rabbitmq/
│   ├── 06-springboot/
│   ├── 07-nodejs/
│   ├── 08-prometheus/
│   ├── 09-node-exporter/
│   └── 10-grafana/
└── terraform/
    └── environments/
        ├── dev/
        ├── staging/
        └── prod/
```

## Environment Specifications

### Development Environment

**Purpose**: Local development and testing

**Characteristics**:

- **Namespace**: `dev`
- **Replicas**: 1 (minimal resources)
- **Storage**: 5Gi (reduced from production)
- **Resource Limits**: Low (CPU: 250m, Memory: 256Mi)
- **Monitoring**: Enabled but basic
- **Updates**: Frequent (RollingUpdate with maxSurge: 1)
- **Database**: Single instance, no replication

**Typical Usage**:

```bash
helm install devops-app ./helm/devops-app -n dev -f helm/devops-app/values-dev.yaml
```

### Staging Environment

**Purpose**: Pre-production testing and validation

**Characteristics**:

- **Namespace**: `staging`
- **Replicas**: 2 (medium resources)
- **Storage**: 20Gi
- **Resource Limits**: Medium (CPU: 500m, Memory: 512Mi)
- **Monitoring**: Fully enabled
- **Updates**: Conservative (RollingUpdate with maxSurge: 1, maxUnavailable: 0)
- **Database**: Single instance with backups
- **Network Policies**: Basic security rules

**Typical Usage**:

```bash
helm install devops-app ./helm/devops-app -n staging -f helm/devops-app/values-staging.yaml
```

### Production Environment

**Purpose**: Live production workloads

**Characteristics**:

- **Namespace**: `production`
- **Replicas**: 3 (high availability)
- **Storage**: 50Gi (large for production data)
- **Resource Limits**: High (CPU: 1000m, Memory: 1Gi)
- **Monitoring**: Comprehensive with detailed alerting
- **Updates**: Zero-downtime (RollingUpdate: maxSurge: 1, maxUnavailable: 0)
- **Database**: Replicated setup with failover
- **Network Policies**: Strict security rules
- **Pod Disruption Budgets**: Enabled for high availability

**Typical Usage**:

```bash
helm install devops-app ./helm/devops-app -n production -f helm/devops-app/values-prod.yaml
```

## Key Differences Between Environments

| Feature | Development | Staging | Production |
|---------|-------------|---------|------------|
| **Replicas** | 1 | 2 | 3 |
| **CPU Request** | 250m | 500m | 500m |
| **CPU Limit** | 500m | 1000m | 1000m |
| **Memory Request** | 256Mi | 512Mi | 512Mi |
| **Memory Limit** | 512Mi | 1Gi | 1Gi |
| **Storage Size** | 5Gi | 20Gi | 50Gi |
| **DB Backups** | None | Daily | Hourly |
| **Monitoring** | Basic | Full | Full + Alerting |
| **Network Policies** | None | Basic | Strict |
| **SLA** | None | 99% | 99.9% |
| **Auto-scaling** | Disabled | Disabled | Enabled |

## Configuration Management with Helm

Each environment uses Helm for managing different configurations.

### Base Configuration

Common values shared across all environments are in `helm/devops-app/values.yaml`.

### Environment Overrides

Environment-specific customizations use Helm values files (values-dev.yaml, values-staging.yaml, values-prod.yaml).

**Example: helm/devops-app/values-dev.yaml**

```yaml
replicaCount: 1

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

postgresql:
  enabled: true
  persistence:
    size: 5Gi

springboot:
  replicaCount: 1
  
nodejs:
  replicaCount: 1

prometheus:
  enabled: true
  retention: 7d

labels:
  environment: development
  managed-by: helm
```

## Deployment Workflow

### Deploying to Development

```bash
# Review what will be deployed
helm template devops-app ./helm/devops-app -n dev -f helm/devops-app/values-dev.yaml | less

# Deploy
helm install devops-app ./helm/devops-app -n dev -f helm/devops-app/values-dev.yaml

# Verify deployment
kubectl get all -n dev
helm status devops-app -n dev
```

### Deploying to Staging

```bash
# Test the staging manifests with dry-run
helm upgrade --install devops-app ./helm/devops-app -n staging -f helm/devops-app/values-staging.yaml --dry-run

# Deploy to staging
helm upgrade --install devops-app ./helm/devops-app -n staging -f helm/devops-app/values-staging.yaml

# Verify
kubectl get all -n staging
helm status devops-app -n staging
```

### Deploying to Production

```bash
# Carefully review production manifests
helm template devops-app ./helm/devops-app -n production -f helm/devops-app/values-prod.yaml | less

# Dry-run to ensure no issues
helm upgrade --install devops-app ./helm/devops-app -n production -f helm/devops-app/values-prod.yaml --dry-run

# Deploy with approval process
helm upgrade --install devops-app ./helm/devops-app -n production -f helm/devops-app/values-prod.yaml

# Monitor deployment
kubectl rollout status deployment/springboot-backend -n production
kubectl rollout status deployment/nodejs-frontend -n production
helm status devops-app -n production
```

## Secret Management by Environment

### Development

Use test credentials:

```bash
helm install devops-app ./helm/devops-app -n dev \
  -f helm/devops-app/values-dev.yaml \
  --set secrets.postgresql.password=dev_password
```

### Staging

Use staging credentials from vault:

```bash
# Example: HashiCorp Vault
vault kv get secret/staging/springboot

# Install with Vault-managed secrets
helm install devops-app ./helm/devops-app -n staging \
  -f helm/devops-app/values-staging.yaml
```

### Production

Use production credentials from secure vault:

```bash
# NEVER hardcode production secrets
# Use: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, Sealed Secrets

helm install devops-app ./helm/devops-app -n production \
  -f helm/devops-app/values-prod.yaml
```

## Monitoring and Logging by Environment

### Development

- Basic Prometheus scraping
- Simple Grafana dashboards
- Logs retained for 7 days

### Staging

- Full Prometheus monitoring
- Pre-production dashboards
- Logs retained for 30 days
- Alert rules testing

### Production

- Comprehensive Prometheus setup
- Production-grade Grafana dashboards
- Logs retained for 90 days
- Active alerting enabled
- Integration with PagerDuty/Slack
- Dynatrace APM enabled

## Scaling Considerations

### Development

- Manual scaling only
- Single node cluster acceptable
- Resource sharing encouraged

### Staging

- Manual scaling with testing
- Multi-node cluster recommended
- Can simulate production load

### Production

- Horizontal Pod Autoscaling (HPA) enabled (2-10 replicas)
- Vertical Pod Autoscaling (VPA) for right-sizing
- Multi-zone/multi-cluster setup for disaster recovery

## Cost Optimization by Environment

| Environment | Estimated Monthly Cost | Optimization |
|---|---|---|
| Development | $50-100 | Shared resources, spot instances |
| Staging | $200-300 | Reserved instances, cost limits |
| Production | $500-1000+ | RI, spot for batch jobs, CDN |

## Backup and Disaster Recovery

### Development

- No backup required
- Data loss acceptable

### Staging

- Daily backups
- 7-day retention
- RTO: 4 hours, RPO: 1 day

### Production

- Hourly backups
- 30-day retention (or compliance requirement)
- RTO: 15 minutes, RPO: 1 hour
- Multi-region failover setup

## Helm Commands Reference

### Installation

```bash
# Install fresh release
helm install devops-app ./helm/devops-app -n <env> -f helm/devops-app/values-<env>.yaml

# Upgrade existing release
helm upgrade --install devops-app ./helm/devops-app -n <env> -f helm/devops-app/values-<env>.yaml
```

### Management

```bash
# List releases
helm list -n <env>

# Get release status
helm status devops-app -n <env>

# Get release values
helm get values devops-app -n <env>

# Get release history
helm history devops-app -n <env>

# Rollback to previous version
helm rollback devops-app <revision> -n <env>
```

### Debugging

```bash
# Template rendering (dry-run)
helm template devops-app ./helm/devops-app -n <env> -f helm/devops-app/values-<env>.yaml

# Render with Helm upgrade (another dry-run)
helm upgrade --install devops-app ./helm/devops-app -n <env> -f helm/devops-app/values-<env>.yaml --dry-run

# Debug template rendering
helm template devops-app ./helm/devops-app -n <env> -f helm/devops-app/values-<env>.yaml --debug
```

## Best Practices

1. ✅ **Never promote directly to production** - Always test in staging first
2. ✅ **Use separate Kubernetes clusters** - Each environment on different cluster
3. ✅ **Manage secrets securely** - Use vault solutions, never hardcode
4. ✅ **Version control everything** - GitOps approach for traceability
5. ✅ **Environment-specific values** - Use Helm values files for overrides
6. ✅ **Monitoring from day one** - Track metrics in all environments
7. ✅ **Backup policies** - Automated backups with testing
8. ✅ **Documentation** - Keep environment specs documented
9. ✅ **Chart versioning** - Use semantic versioning for Helm charts
10. ✅ **Use Helm hooks** - For pre/post deployment tasks

## Support and Troubleshooting

For environment-specific issues:

```bash
# Check environment namespace
kubectl get ns | grep -E 'dev|staging|production'

# Get environment-specific resources
kubectl get all -n dev
kubectl get all -n staging
kubectl get all -n production

# View environment-specific logs
kubectl logs -f deployment/springboot-backend -n dev
kubectl logs -f deployment/springboot-backend -n staging
kubectl logs -f deployment/springboot-backend -n production

# Get Helm release info
helm status devops-app -n dev
helm values devops-app -n dev
```

## Contact

For questions about environment setup or deployment strategies, refer to the main README.md or create an issue in the repository.
