# NodePulse - End-to-End Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [Accessing Components](#accessing-components)
5. [Monitoring Setup](#monitoring-setup)
6. [Security Configuration](#security-configuration)
7. [Troubleshooting](#troubleshooting)
8. [Upgrading](#upgrading)
9. [Uninstalling](#uninstalling)

## Prerequisites

### 1. Kubernetes Cluster
- Kubernetes version 1.16 or later
- Minimum cluster resources:
  - 4 CPU cores
  - 8GB RAM
  - 100GB storage
- StorageClass configured for dynamic provisioning

### 2. Required Tools
```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install Helm
curl -fsSL -o get-helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get-helm.sh
./get-helm.sh
```

### 3. Cluster Setup Verification
```bash
# Verify kubectl access
kubectl cluster-info

# Verify Helm installation
helm version

# Check cluster resources
kubectl get nodes
kubectl describe nodes
```

## Installation

### 1. Add NodePulse Repository
```bash
# Add the Helm repository
helm repo add nodepulse https://github.com/DARK-art108/nodepulse/releases/download/v0.1.0

# Update repositories
helm repo update

# Verify repository addition
helm repo list
```

### 2. Create Monitoring Namespace
```bash
# Create namespace
kubectl create namespace monitoring

# Verify namespace creation
kubectl get namespace monitoring
```

### 3. Install NodePulse
```bash
# Basic installation
helm install nodepulse nodepulse/nodepulse -n monitoring

# Verify installation
helm list -n monitoring
kubectl get pods -n monitoring -w
```

## Configuration

### 1. Basic Configuration
```bash
# Install with custom values
helm install nodepulse nodepulse/nodepulse -n monitoring \
  --set prometheus.server.retention=30d \
  --set grafana.adminPassword=your-secure-password \
  --set loki.persistence.size=100Gi \
  --set trivy-operator.vulnerabilityScanner.schedule="0 */6 * * *"
```

### 2. Advanced Configuration
Create a `values.yaml` file:
```yaml
prometheus:
  server:
    retention: 30d
    resources:
      requests:
        memory: 2Gi
        cpu: 500m
      limits:
        memory: 4Gi
        cpu: 1000m

grafana:
  adminPassword: your-secure-password
  resources:
    requests:
      memory: 256Mi
      cpu: 100m
    limits:
      memory: 512Mi
      cpu: 200m

loki:
  persistence:
    size: 100Gi

trivy-operator:
  vulnerabilityScanner:
    schedule: "0 */6 * * *"
```

Install using the values file:
```bash
helm install nodepulse nodepulse/nodepulse -n monitoring -f values.yaml
```

## Accessing Components

### 1. Grafana Dashboard
```bash
# Port-forward Grafana service
kubectl port-forward -n monitoring svc/nodepulse-grafana 3000:80

# Access Grafana
# URL: http://localhost:3000
# Username: admin
# Password: (set during installation)
```

### 2. Prometheus UI
```bash
# Port-forward Prometheus service
kubectl port-forward -n monitoring svc/nodepulse-prometheus-server 9090:9090

# Access Prometheus
# URL: http://localhost:9090
```

### 3. Loki (Logs)
```bash
# Port-forward Loki service
kubectl port-forward -n monitoring svc/nodepulse-loki 3100:3100

# Access Loki
# URL: http://localhost:3100
```

## Monitoring Setup

### 1. Configure Service Discovery
```yaml
# Example service monitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: example-service
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: example
  endpoints:
  - port: web
    interval: 30s
```

### 2. Set Up Alerts
```yaml
# Example alert rule
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

### 3. Configure Grafana Dashboards
1. Access Grafana UI
2. Navigate to Dashboards
3. Import pre-configured dashboards:
   - Kubernetes / Compute Resources / Cluster
   - Kubernetes / Compute Resources / Namespace (Pods)
   - Kubernetes / Compute Resources / Pod
   - Kubernetes / Compute Resources / Workload

## Security Configuration

### 1. Network Policies
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

### 2. RBAC Configuration
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: monitoring-role
  namespace: monitoring
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
```

### 3. TLS Configuration
```bash
# Enable TLS for components
helm upgrade nodepulse nodepulse/nodepulse -n monitoring \
  --set prometheus.server.ingress.tls.enabled=true \
  --set grafana.ingress.tls.enabled=true \
  --set loki.ingress.tls.enabled=true
```

## Troubleshooting

### 1. Check Component Status
```bash
# Check all pods
kubectl get pods -n monitoring

# Check specific component
kubectl describe pod -n monitoring <pod-name>

# Check logs
kubectl logs -n monitoring <pod-name>
```

### 2. Storage Issues
```bash
# Check PVC status
kubectl get pvc -n monitoring

# Check storage class
kubectl get storageclass

# Check persistent volumes
kubectl get pv
```

### 3. Network Issues
```bash
# Check services
kubectl get svc -n monitoring

# Check endpoints
kubectl get endpoints -n monitoring

# Test connectivity
kubectl run -it --rm --restart=Never --image=curlimages/curl curl -- \
  curl http://nodepulse-prometheus-server.monitoring:9090/-/healthy
```

## Upgrading

### 1. Check Current Version
```bash
helm list -n monitoring
```

### 2. Update Repository
```bash
helm repo update
```

### 3. Upgrade Release
```bash
# Basic upgrade
helm upgrade nodepulse nodepulse/nodepulse -n monitoring

# Upgrade with custom values
helm upgrade nodepulse nodepulse/nodepulse -n monitoring -f values.yaml
```

### 4. Verify Upgrade
```bash
helm history nodepulse -n monitoring
kubectl get pods -n monitoring
```

## Uninstalling

### 1. Uninstall NodePulse
```bash
helm uninstall nodepulse -n monitoring
```

### 2. Clean Up Resources
```bash
# Delete namespace (optional)
kubectl delete namespace monitoring

# Delete PVCs (if needed)
kubectl delete pvc -n monitoring --all
```

### 3. Remove Repository
```bash
helm repo remove nodepulse
```

## Next Steps

1. Configure persistent storage for production
2. Set up proper authentication
3. Configure custom dashboards
4. Set up alerting rules
5. Configure backup and restore procedures

For more information, visit the [NodePulse Documentation](https://github.com/DARK-art108/nodepulse/blob/main/README.md). 