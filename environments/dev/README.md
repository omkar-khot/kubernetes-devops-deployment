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
```bash
kubectl apply -k environments/dev/
```

### Using Helm
```bash
helm install devops-app ./helm/devops-app \
  -n dev \
  --values helm/devops-app/values-dev.yaml
```

### Using Terraform
```bash
terraform apply -var-file=environments/dev/terraform.tfvars
```

## Access Services

### Spring Boot Backend
```bash
kubectl port-forward svc/springboot-service 8080:8080 -n dev
# Access: http://localhost:8080
```

### React Frontend
```bash
kubectl port-forward svc/react-frontend 3000:3000 -n dev
# Access: http://localhost:3000
```

### Prometheus
```bash
kubectl port-forward svc/prometheus 9090:9090 -n dev
# Access: http://localhost:9090
```

### Dynatrace OneAgent
Automatically deployed via DaemonSet for development monitoring.

## Secrets Management

### Retrieve dev secrets from Vault
```bash
vault kv get secret/dev/springboot
vault kv get secret/dev/postgres
vault kv get secret/dev/rabbitmq
```

### Update secrets in Vault
```bash
vault kv put secret/dev/springboot \
  DB_PASSWORD=dev_password \
  RABBITMQ_PASSWORD=dev_rabbitmq
```

## Logging with ELK

### View Spring Boot logs
```bash
kubectl logs -f deployment/springboot-backend -n dev
```

### Access Kibana for centralized logs
```bash
kubectl port-forward svc/kibana 5601:5601 -n dev
# Access: http://localhost:5601
```

### Query logs in Kibana
```
environment:dev AND level:ERROR
service:springboot AND timestamp:[now-1h TO now]
```

## Database Access

### PostgreSQL
```bash
kubectl exec -it postgres-0 -n dev -- psql -U postgres -d mydb
```

### RabbitMQ Management UI
```bash
kubectl port-forward svc/rabbitmq-mgmt 15672:15672 -n dev
# Access: http://localhost:15672 (guest/guest)
```

## Troubleshooting

### Check pod status
```bash
kubectl get pods -n dev
kubectl describe pod <pod-name> -n dev
```

### View logs
```bash
kubectl logs <pod-name> -n dev
```

### Check resource usage
```bash
kubectl top pods -n dev
```

## CI/CD Pipeline

Development environment is automatically updated on every push to `main` branch via GitHub Actions workflow: `.github/workflows/deploy-dev.yml`

### Workflow Steps:
1. Code pushed to GitHub
2. GitHub Actions triggers automated tests
3. Docker images built and pushed to ECR
4. Helm charts deployed to dev namespace
5. Smoke tests run
6. Dynatrace captures baseline metrics

## Best Practices

- ✅ Frequently test changes before promotion to staging
- ✅ Use consistent namespace (`dev`) for all resources
- ✅ Monitor resource usage via Dynatrace
- ✅ Check logs in ELK for errors
- ✅ Clean up old pods and PVCs periodically
- ✅ Never store production secrets in dev

## Maintenance

- **Daily**: Monitor Dynatrace dashboard for anomalies
- **Weekly**: Review and clean up test data
- **Monthly**: Update base images in Dockerfile
- **Quarterly**: Review and update resource limits

## Support

For issues, check logs in Kibana or contact the DevOps team.
