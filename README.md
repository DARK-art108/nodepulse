# NodePulse 🚀

<div align="center">

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Helm](https://img.shields.io/badge/Helm-3.8.0+-0F1689?logo=helm)](https://helm.sh)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.19+-326CE5?logo=kubernetes)](https://kubernetes.io)
[![Docker](https://img.shields.io/badge/Docker-4.0+-2496ED?logo=docker)](https://www.docker.com)
[![Prometheus](https://img.shields.io/badge/Prometheus-2.45.0-E6522C?logo=prometheus)](https://prometheus.io)
[![Grafana](https://img.shields.io/badge/Grafana-10.0.0-F46800?logo=grafana)](https://grafana.com)
[![Trivy](https://img.shields.io/badge/Trivy-0.45.0-2496ED?logo=aqua)](https://aquasecurity.github.io/trivy)
[![GitHub Release](https://img.shields.io/github/v/release/DARK-art108/nodepulse?include_prereleases&sort=semver)](https://github.com/DARK-art108/nodepulse/releases)
[![GitHub Stars](https://img.shields.io/github/stars/DARK-art108/nodepulse?style=social)](https://github.com/DARK-art108/nodepulse/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/DARK-art108/nodepulse?style=social)](https://github.com/DARK-art108/nodepulse/network/members)
[![GitHub Issues](https://img.shields.io/github/issues/DARK-art108/nodepulse)](https://github.com/DARK-art108/nodepulse/issues)
[![GitHub Pull Requests](https://img.shields.io/github/issues-pr/DARK-art108/nodepulse)](https://github.com/DARK-art108/nodepulse/pulls)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/nodepulse)](https://artifacthub.io/packages/helm/nodepulse/nodepulse)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/DARK-art108/nodepulse/graphs/commit-activity)
[![Documentation](https://img.shields.io/badge/Documentation-Complete-brightgreen)](https://github.com/DARK-art108/nodepulse/docs)
[![CI/CD](https://img.shields.io/badge/CI/CD-Passing-brightgreen)](https://github.com/DARK-art108/nodepulse/actions)
[![Code Quality](https://img.shields.io/badge/Code%20Quality-A+-brightgreen)](https://github.com/DARK-art108/nodepulse/actions)
[![Security](https://img.shields.io/badge/Security-Scanned-brightgreen)](https://github.com/DARK-art108/nodepulse/security)
[![Dependencies](https://img.shields.io/badge/Dependencies-Up%20to%20date-brightgreen)](https://github.com/DARK-art108/nodepulse/network/dependencies)

</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/DARK-art108/nodepulse/main/docs/architecture.png" alt="NodePulse Architecture" width="800"/>
</div>

## ✨ Overview

NodePulse is a powerful, all-in-one Kubernetes observability stack that brings together the best of Prometheus, Grafana, and Trivy to deliver comprehensive monitoring and security insights for your Kubernetes clusters.

## 🚀 Quick Start

### Prerequisites

| Environment | Requirements |
|------------|-------------|
| **Docker Desktop** | <ul><li>Docker Desktop 4.0+</li><li>Kubernetes enabled</li><li>4GB+ RAM</li><li>2+ CPU cores</li></ul> |
| **Production** | <ul><li>Kubernetes 1.19+</li><li>Helm 3.8.0+</li><li>kubectl</li><li>Persistent storage</li></ul> |

### Installation

```bash
# Add Helm repositories
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add aquasecurity https://aquasecurity.github.io/helm-charts
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install NodePulse
helm install nodepulse . -n monitoring
```

## 🛠️ Features

### 📊 Prometheus
- **Metrics Collection**
  - Automatic service discovery
  - Custom metric support
  - 5-day data retention
  - Configurable intervals
  - Built-in alert rules

- **Alerting**
  - Node resource monitoring
  - Pod health tracking
  - Container limits
  - Service availability

### 📈 Grafana
- **Dashboards**
  - Cluster overview
  - Node metrics
  - Pod metrics
  - Service metrics
  - Resource utilization

- **Data Sources**
  - Prometheus integration
  - Additional datasources
  - Secure proxy access

### 🔒 Trivy
- **Security Scanning**
  - Daily vulnerability checks
  - Configuration audits
  - Secret scanning
  - Custom schedules
  - Report retention

## 📋 Configuration

### Resource Requirements

| Component | Docker Desktop | Production |
|-----------|---------------|------------|
| **Prometheus** | <ul><li>1Gi RAM</li><li>500m CPU</li></ul> | <ul><li>4Gi RAM</li><li>2 CPU</li></ul> |
| **Grafana** | <ul><li>256Mi RAM</li><li>100m CPU</li></ul> | <ul><li>1Gi RAM</li><li>500m CPU</li></ul> |
| **Trivy** | <ul><li>512Mi RAM</li><li>100m CPU</li></ul> | <ul><li>2Gi RAM</li><li>1 CPU</li></ul> |

### Storage Requirements

| Component | Docker Desktop | Production |
|-----------|---------------|------------|
| **Prometheus** | 10Gi | 50Gi+ |
| **Grafana** | 5Gi | 20Gi+ |
| **Trivy Reports** | 1Gi | 10Gi+ |

## 🔗 Accessing Components

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

## 📊 Monitoring Capabilities

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

## 🛠️ Example Dashboards

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

## 🔍 Troubleshooting

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

## 🔒 Security Best Practices

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

## 🤝 Support

<div align="center">

[![GitHub Issues](https://img.shields.io/badge/GitHub-Issues-blue?logo=github)](https://github.com/DARK-art108/nodepulse/issues)
[![Documentation](https://img.shields.io/badge/Documentation-Read-blue?logo=read-the-docs)](https://github.com/DARK-art108/nodepulse/docs)
[![Community](https://img.shields.io/badge/Community-Discussions-green?logo=discourse)](https://github.com/DARK-art108/nodepulse/discussions)

</div>

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">
  <sub>Built with ❤️ by the NodePulse Team</sub>
</div> 