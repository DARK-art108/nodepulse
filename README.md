# NodePulse

NodePulse is a plug-and-play observability stack for Kubernetes that combines Prometheus, Grafana, and Trivy to deliver unified metrics and security insights.

![NodePulse Architecture](https://raw.githubusercontent.com/DARK-art108/nodepulse/main/docs/architecture.png)

## Features

| Component | Features | Benefits |
|-----------|----------|----------|
| **Prometheus** | - Metrics collection<br>- Alerting<br>- Service discovery<br>- Time series database | - Real-time monitoring<br>- Custom alerts<br>- Historical data analysis |
| **Grafana** | - Dashboard visualization<br>- Alert management<br>- Data source integration | - Beautiful visualizations<br>- Custom dashboards<br>- Team collaboration |
| **Trivy** | - Vulnerability scanning<br>- Security reporting<br>- Daily automated scans | - Security compliance<br>- Risk assessment<br>- Proactive security |

## Quick Start

### Prerequisites

| Environment | Requirements |
|------------|-------------|
| **Docker Desktop** | - Docker Desktop 4.0+<br>- Kubernetes enabled<br>- 4GB+ RAM<br>- 2+ CPU cores |
| **Production** | - Kubernetes 1.19+<br>- Helm 3.8.0+<br>- kubectl<br>- Persistent storage |

### Installation

1. Add Helm repositories:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add aquasecurity https://aquasecurity.github.io/helm-charts
helm repo update
```

2. Create monitoring namespace:
```bash
kubectl create namespace monitoring
```

3. Install NodePulse:
```bash
helm install nodepulse . -n monitoring
```

## Configuration Guide

### Resource Requirements

| Component | Docker Desktop | Production |
|-----------|---------------|------------|
| **Prometheus** | 1Gi RAM, 500m CPU | 4Gi RAM, 2 CPU |
| **Grafana** | 256Mi RAM, 100m CPU | 1Gi RAM, 500m CPU |
| **Trivy** | 512Mi RAM, 100m CPU | 2Gi RAM, 1 CPU |

### Storage Requirements

| Component | Docker Desktop | Production |
|-----------|---------------|------------|
| **Prometheus** | 10Gi | 50Gi+ |
| **Grafana** | 5Gi | 20Gi+ |
| **Trivy Reports** | 1Gi | 10Gi+ |

## Accessing Components

### Docker Desktop

| Component | Access Method | Default Credentials |
|-----------|--------------|---------------------|
| **Grafana** | http://localhost:3000 | admin/admin |
| **Prometheus** | http://localhost:9090 | - |
| **AlertManager** | http://localhost:9093 | - |

### Production

| Component | Access Method | Security Notes |
|-----------|--------------|----------------|
| **Grafana** | https://your-domain/grafana | Configure SSO |
| **Prometheus** | https://your-domain/prometheus | Enable TLS |
| **AlertManager** | https://your-domain/alertmanager | Enable TLS |

## Monitoring Capabilities

### System Metrics

| Metric Type | Description | Collection Method |
|------------|-------------|-------------------|
| **Cluster State** | Node status, pod states | kube-state-metrics |
| **Resource Usage** | CPU, memory, network | Prometheus |
| **Application Metrics** | Custom metrics | Prometheus client |

### Security Scanning

| Scan Type | Frequency | Report Location |
|-----------|-----------|-----------------|
| **Vulnerability** | Daily | Trivy Dashboard |
| **Configuration** | Continuous | Trivy Dashboard |
| **Compliance** | Weekly | Trivy Reports |

## Example Dashboards

1. **Cluster Overview**
   - Node status
   - Resource usage
   - Pod distribution

2. **Application Health**
   - Service status
   - Error rates
   - Response times

3. **Security Dashboard**
   - Vulnerability status
   - Risk assessment
   - Compliance score

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| **Prometheus OOM** | Increase memory limits |
| **Grafana Login Issues** | Reset admin password |
| **Trivy Scan Failures** | Check resource limits |

### Logs Location

| Component | Log Command |
|-----------|-------------|
| **Prometheus** | `kubectl logs -n monitoring -l app.kubernetes.io/name=prometheus` |
| **Grafana** | `kubectl logs -n monitoring -l app.kubernetes.io/name=grafana` |
| **Trivy** | `kubectl logs -n monitoring -l app.kubernetes.io/name=trivy-operator` |

## Security Best Practices

1. **Authentication**
   - Enable SSO for Grafana
   - Configure RBAC
   - Use service accounts

2. **Network Security**
   - Enable TLS
   - Configure network policies
   - Use private networks

3. **Data Protection**
   - Encrypt sensitive data
   - Regular backups
   - Access controls

## Support

- GitHub Issues: https://github.com/DARK-art108/nodepulse/issues
- Documentation: https://github.com/DARK-art108/nodepulse/docs
- Community: https://github.com/DARK-art108/nodepulse/discussions

## License

MIT License 