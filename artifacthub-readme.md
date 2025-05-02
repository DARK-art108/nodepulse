# NodePulse Helm Chart

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/nodepulse)](https://artifacthub.io/packages/helm/nodepulse/nodepulse)

## Introduction

NodePulse is a comprehensive Kubernetes observability stack that combines multiple open-source tools to provide a unified monitoring solution. This Helm chart packages Prometheus, Grafana, Loki, Trivy, and OpenTelemetry into a single, easy-to-deploy solution.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.x
- kubectl configured to access your cluster
- Minimum cluster resources:
  - 4 CPU cores
  - 8GB RAM
  - 100GB storage

## Installation

### Add the Helm Repository

```bash
helm repo add nodepulse https://github.com/DARK-art108/nodepulse/releases/download/v0.1.0
helm repo update
```

### Install the Chart

```bash
# Create monitoring namespace
kubectl create namespace monitoring

# Install with default values
helm install nodepulse nodepulse/nodepulse -n monitoring
```

## Configuration

### Key Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `prometheus.server.retention` | Prometheus data retention period | 15d |
| `grafana.adminPassword` | Grafana admin password | admin |
| `loki.persistence.size` | Loki storage size | 50Gi |
| `trivy-operator.vulnerabilityScanner.schedule` | Security scan schedule | 0 0 * * * |

### Custom Installation

```bash
helm install nodepulse nodepulse/nodepulse -n monitoring \
  --set prometheus.server.retention=30d \
  --set grafana.adminPassword=your-secure-password \
  --set loki.persistence.size=100Gi \
  --set trivy-operator.vulnerabilityScanner.schedule="0 */6 * * *"
```

### Resource Configuration

```yaml
# Example custom values.yaml
prometheus:
  server:
    resources:
      requests:
        memory: 2Gi
        cpu: 500m
      limits:
        memory: 4Gi
        cpu: 1000m

grafana:
  resources:
    requests:
      memory: 256Mi
      cpu: 100m
    limits:
      memory: 512Mi
      cpu: 200m
```

## Components

### Prometheus
- Metrics collection and storage
- Alerting rules
- Service discovery
- Default retention: 15 days

### Grafana
- Visualization dashboards
- Alert management
- Default credentials:
  - Username: admin
  - Password: (set during installation)

### Loki
- Log aggregation
- LogQL query language
- Default storage: 50Gi

### Trivy Operator
- Vulnerability scanning
- Configuration auditing
- Default scan schedule: Daily

### OpenTelemetry
- Distributed tracing
- Metrics collection
- Log collection

## Accessing Components

### Grafana Dashboard
```bash
kubectl port-forward -n monitoring svc/nodepulse-grafana 3000:80
```
Access: http://localhost:3000

### Prometheus UI
```bash
kubectl port-forward -n monitoring svc/nodepulse-prometheus-server 9090:9090
```
Access: http://localhost:9090

### Loki
```bash
kubectl port-forward -n monitoring svc/nodepulse-loki 3100:3100
```
Access: http://localhost:3100

## Security

### Authentication
- Grafana: Basic auth with custom password
- Prometheus: No auth by default (use network policies)
- Loki: No auth by default (use network policies)

### Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nodepulse-network-policy
  namespace: monitoring
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: nodepulse
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: monitoring
```

## Monitoring and Alerts

### Pre-configured Alerts
- Node memory pressure
- Pod restarts
- High CPU usage
- Disk space
- Security vulnerabilities

### Custom Alert Rules
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-alerts
  namespace: monitoring
spec:
  groups:
    - name: custom
      rules:
        - alert: HighErrorRate
          expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: High error rate detected
```

## Upgrading

```bash
# Update repositories
helm repo update

# Upgrade release
helm upgrade nodepulse nodepulse/nodepulse -n monitoring
```

## Uninstalling

```bash
# Uninstall the chart
helm uninstall nodepulse -n monitoring

# Delete namespace (optional)
kubectl delete namespace monitoring
```

## Troubleshooting

### Common Issues

1. **Storage Issues**
```bash
# Check PVC status
kubectl get pvc -n monitoring

# Check storage class
kubectl get storageclass
```

2. **Resource Constraints**
```bash
# Check pod status
kubectl get pods -n monitoring

# Check resource usage
kubectl top pods -n monitoring
```

3. **Network Issues**
```bash
# Check service status
kubectl get svc -n monitoring

# Check network policies
kubectl get networkpolicy -n monitoring
```

## Contributing

We welcome contributions! Please see our [Contributing Guide](https://github.com/DARK-art108/nodepulse/blob/main/CONTRIBUTING.md) for details.

## License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/DARK-art108/nodepulse/blob/main/LICENSE) file for details. 