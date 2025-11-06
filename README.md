# Kubernetes DevOps Deployment Package

## Complete Tech Stack Configuration

This repository contains production-ready Kubernetes manifests for a complete DevOps tech stack:

- **Backend**: Spring Boot (Java)
- **Frontend**: Node.js
- **Database**: PostgreSQL
- **Message Broker**: RabbitMQ
- **Monitoring**: Prometheus & Grafana
- **Node Monitoring**: Prometheus Node Exporter

## 📁 Directory Structure

```
├── 01-namespaces/
├── 02-secrets/
├── 03-configmaps/
├── 04-postgres/
├── 05-rabbitmq/
├── 06-springboot/
├── 07-nodejs/
├── 08-prometheus/
├── 09-node-exporter/
└── 10-grafana/
```

## 🚀 Quick Start

### Prerequisites
- Kubernetes cluster (v1.20+)
- kubectl configured
- StorageClass configured for persistent volumes

### Step-by-Step Deployment

```bash
# 1. Create namespace
kubectl create namespace production

# 2. Deploy secrets
kubectl apply -f 02-secrets/ -n production

# 3. Deploy infrastructure
kubectl apply -f 03-configmaps/ -n production
kubectl apply -f 04-postgres/ -n production
kubectl apply -f 05-rabbitmq/ -n production

# 4. Deploy applications
kubectl apply -f 06-springboot/ -n production
kubectl apply -f 07-nodejs/ -n production

# 5. Deploy monitoring
kubectl apply -f 08-prometheus/ -n production
kubectl apply -f 09-node-exporter/ -n production
kubectl apply -f 10-grafana/ -n production
```

## 📊 Components Overview

### 1. **PostgreSQL** (Stateful)
- StatefulSet with persistent storage
- Default credentials in secrets
- Database: mydb

### 2. **RabbitMQ** (Stateful)
- StatefulSet with management UI (port 15672)
- AMQP port: 5672
- Default user/password in secrets

### 3. **Spring Boot Backend** (Stateless)
- Deployment with 3 replicas
- Connects to PostgreSQL and RabbitMQ
- Exposes metrics to Prometheus
- Port: 8080

### 4. **Node.js Frontend** (Stateless)
- Deployment with 3 replicas
- LoadBalancer service
- Port: 3000 (internal), 80 (external)

### 5. **Prometheus** (Monitoring)
- Scrapes metrics from all services
- Pre-configured service discovery
- Port: 9090

### 6. **Node Exporter** (DaemonSet)
- Runs on every node
- Collects node-level metrics
- Port: 9100

### 7. **Grafana** (Visualization)
- Pre-configured Prometheus datasource
- Default credentials: admin/admin123
- Port: 3000

## 🔐 Security Notes

- Update all passwords in secrets before production
- Use strong credentials (not base64 encoded defaults)
- Consider using sealed-secrets or similar for production
- Enable RBAC for access control
- Use network policies to restrict traffic

## 📝 Customization

### Update Docker Images
```bash
sed -i 's/your-registry/myregistry.azurecr.io/g' 06-springboot/*.yaml
sed -i 's/your-registry/myregistry.azurecr.io/g' 07-nodejs/*.yaml
```

### Change Resource Limits
Edit the resource requests/limits in deployment YAML files based on your cluster capacity.

### Update Passwords
Edit secrets in `02-secrets/` directory and base64 encode your passwords.

## 🔍 Accessing Services

### Get LoadBalancer IPs
```bash
kubectl get svc -n production
```

### Port Forwarding
```bash
# Grafana
kubectl port-forward svc/grafana-service 3000:3000 -n production

# Prometheus
kubectl port-forward svc/prometheus-service 9090:9090 -n production

# Spring Boot
kubectl port-forward svc/springboot-service 8080:8080 -n production
```

## 📊 Monitoring Setup

1. Access Grafana: http://localhost:3000 (admin/admin123)
2. Add Prometheus datasource: http://prometheus-service:9090
3. Import dashboards for Node Exporter and custom metrics

## 🛠️ Troubleshooting

```bash
# Check pod status
kubectl get pods -n production

# Describe pod for errors
kubectl describe pod <pod-name> -n production

# View logs
kubectl logs -f deployment/<deployment-name> -n production

# Check services
kubectl get svc -n production

# Check persistent volumes
kubectl get pvc -n production
```

## ✅ Deployment Checklist

- [ ] Update all password credentials in secrets
- [ ] Update Docker image registry names
- [ ] Configure StorageClass if needed
- [ ] Verify cluster has enough resources
- [ ] Deploy namespace first
- [ ] Deploy secrets and configmaps
- [ ] Deploy infrastructure components (DB, message broker)
- [ ] Verify infrastructure is running
- [ ] Deploy applications
- [ ] Deploy monitoring
- [ ] Test end-to-end connectivity
- [ ] Configure ingress if needed
- [ ] Set up backup policies

## 📚 Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Spring Boot on Kubernetes](https://spring.io/guides/kubernetes/)
- [PostgreSQL Operator](https://postgres-operator.readthedocs.io/)
- [RabbitMQ Kubernetes Deployment](https://www.rabbitmq.com/kubernetes/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)

## 📄 License

MIT License - Feel free to use and modify for your projects

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

**Last Updated**: November 2025
**Version**: 1.0
