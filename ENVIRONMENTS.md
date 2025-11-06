# Multi-Environment Kubernetes Deployment

## Overview

This repository now supports **three separate Kubernetes environments**:
- **Development** - For developers testing new features
- **Staging** - Pre-production testing environment
- **Production** - Live production environment

## Directory Structure

The repository is organized by environment:

```
kubernetes-devops-deployment/
├── README.md                              # Main README
├── ENVIRONMENTS.md                        # This file
├── environments/
│   ├── development/
│   │   ├── 01-namespaces/
│   │   ├── 02-secrets/
│   │   ├── 03-configmaps/
│   │   ├── 04-postgres/
│   │   ├── 05-rabbitmq/
│   │   ├── 06-springboot/
│   │   ├── 07-nodejs/
│   │   ├── 08-prometheus/
│   │   ├── 09-node-exporter/
│   │   ├── 10-grafana/
│   │   └── kustomization.yaml
│   │
│   ├── staging/
│   │   ├── 01-namespaces/
│   │   ├── 02-secrets/
│   │   ├── 03-configmaps/
│   │   ├── 04-postgres/
│   │   ├── 05-rabbitmq/
│   │   ├── 06-springboot/
│   │   ├── 07-nodejs/
│   │   ├── 08-prometheus/
│   │   ├── 09-node-exporter/
│   │   ├── 10-grafana/
│   │   └── kustomization.yaml
│   │
│   └── production/
│       ├── 01-namespaces/
│       ├── 02-secrets/
│       ├── 03-configmaps/
│       ├── 04-postgres/
│       ├── 05-rabbitmq/
│       ├── 06-springboot/
│       ├── 07-nodejs/
│       ├── 08-prometheus/
│       ├── 09-node-exporter/
│       ├── 10-grafana/
│       └── kustomization.yaml
│
└── base/                                  # Shared base manifests
    ├── 01-namespaces/
    ├── 02-secrets/
    ├── 03-configmaps/
    ├── 04-postgres/
    ├── 05-rabbitmq/
    ├── 06-springboot/
    ├── 07-nodejs/
    ├── 08-prometheus/
    ├── 09-node-exporter/
    ├── 10-grafana/
    └── kustomization.yaml
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
kubectl apply -k environments/development/
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
kubectl apply -k environments/staging/
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
kubectl apply -k environments/production/
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

## Configuration Management with Kustomize

Each environment uses Kustomize for managing different configurations:

### Base Configuration
Common manifests shared across all environments are in the `base/` directory.

### Environment Overlays
Environment-specific customizations use Kustomize overlays:

```yaml
# environments/development/kustomization.yaml
namespaces:
- dev

bases:
- ../../base

patchesStrategicMerge:
- patches/namespace.yaml
- patches/replicas.yaml
- patches/resources.yaml

replicas:
- name: springboot-backend
  count: 1
- name: nodejs-frontend
  count: 1

commonLabels:
  environment: development
  managed-by: kustomize
```

## Deployment Workflow

### Deploying to Development

```bash
# Navigate to development environment
cd environments/development

# Review what will be deployed
kubectl kustomize . | less

# Deploy
kubectl apply -k .

# Verify deployment
kubectl get all -n dev
```

### Deploying to Staging

```bash
# Test the staging manifests
kubectl apply -k environments/staging/ --dry-run=client

# Deploy to staging
kubectl apply -k environments/staging/

# Verify
kubectl get all -n staging
```

### Deploying to Production

```bash
# Carefully review production manifests
kubectl kustomize environments/production/ | less

# Dry-run to ensure no issues
kubectl apply -k environments/production/ --dry-run=client -o yaml | less

# Deploy with approval process
kubectl apply -k environments/production/

# Monitor deployment
kubectl rollout status deployment/springboot-backend -n production
kubectl rollout status deployment/nodejs-frontend -n production
```

## Secret Management by Environment

### Development
```bash
# Use test credentials
echo -n 'dev_user' | base64
echo -n 'dev_password' | base64
```

### Staging
```bash
# Use staging credentials from secrets manager
# Example: HashiCorp Vault, AWS Secrets Manager
```

### Production
```bash
# Use production credentials from secure vault
# NEVER hardcode production secrets
# Use: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, Sealed Secrets
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

## GitOps Workflow (Optional)

For automated deployments using ArgoCD:

```yaml
# applications/dev-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: devops-deployment-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/omkar-khot/kubernetes-devops-deployment
    targetRevision: main
    path: environments/development
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

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
- Horizontal Pod Autoscaling (HPA) enabled
- Vertical Pod Autoscaling (VPA) for right-sizing
- Multi-zone/multi-cluster setup for disaster recovery

## Cost Optimization by Environment

| Environment | Estimated Monthly Cost | Optimization |
|-------------|----------------------|---------------|
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

## Best Practices

1. ✅ **Never promote directly to production** - Always test in staging first
2. ✅ **Use separate Kubernetes clusters** - Each environment on different cluster
3. ✅ **Manage secrets securely** - Use vault solutions, never hardcode
4. ✅ **Version control everything** - GitOps approach for traceability
5. ✅ **Environment-specific values** - Use Kustomize or Helm for overrides
6. ✅ **Monitoring from day one** - Track metrics in all environments
7. ✅ **Backup policies** - Automated backups with testing
8. ✅ **Documentation** - Keep environment specs documented

## Support and Troubleshooting

For environment-specific issues:

```bash
# Check environment namespace
kubectl get ns | grep -E 'dev|staging|production'

# Get environment-specific resources
kubectl get all -n dev
kubectl get all -n staging
kubectl get all -n production

# Check environment configurations
kubectl get cm -n dev
kubectl get secrets -n dev

# View environment-specific logs
kubectl logs -f deployment/springboot-backend -n dev
kubectl logs -f deployment/springboot-backend -n staging
kubectl logs -f deployment/springboot-backend -n production
```

## Contact

For questions about environment setup or deployment strategies, refer to the main README.md or create an issue in the repository.
