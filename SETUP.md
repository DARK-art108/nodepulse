# NodePulse Setup Guide

This guide provides detailed instructions for setting up NodePulse in both Docker Desktop and production environments.

## Table of Contents

1. [Environment Setup](#environment-setup)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [Monitoring Setup](#monitoring-setup)
5. [Security Configuration](#security-configuration)
6. [Troubleshooting](#troubleshooting)

## Environment Setup

### Docker Desktop

1. **System Requirements**
   ```bash
   # Check Docker version
   docker --version
   # Check Kubernetes status
   kubectl version
   ```

2. **Resource Allocation**
   - Minimum: 4GB RAM, 2 CPU cores
   - Recommended: 8GB RAM, 4 CPU cores
   - Storage: 20GB free space

3. **Enable Kubernetes**
   ```bash
   # Open Docker Desktop
   # Go to Settings > Kubernetes
   # Check "Enable Kubernetes"
   # Click "Apply & Restart"
   ```

### Production Cluster

1. **Prerequisites**
   ```bash
   # Check Kubernetes version
   kubectl version
   # Check Helm version
   helm version
   ```

2. **Resource Requirements**
   - Minimum: 8GB RAM, 4 CPU cores per node
   - Recommended: 16GB RAM, 8 CPU cores per node
   - Storage: 100GB+ per node

3. **Storage Setup**
   ```bash
   # Check available storage classes
   kubectl get storageclass
   # Create storage class if needed
   kubectl apply -f storage-class.yaml
   ```

## Installation

### Docker Desktop

1. **Add Helm Repositories**
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo add grafana https://grafana.github.io/helm-charts
   helm repo add aquasecurity https://aquasecurity.github.io/helm-charts
   helm repo update
   ```

2. **Create Namespace**
   ```bash
   kubectl create namespace monitoring
   ```

3. **Install NodePulse**
   ```bash
   helm install nodepulse . -n monitoring
   ```

### Production

1. **Custom Values**
   ```bash
   # Create custom values file
   cat > values-production.yaml << EOF
   global:
     storageClass: standard
   
   prometheus:
     server:
       persistentVolume:
         size: 50Gi
       retention: 15d
       resources:
         limits:
           memory: 4Gi
           cpu: 2
   
   grafana:
     persistence:
       size: 20Gi
     resources:
       limits:
         memory: 1Gi
         cpu: 500m
   
   trivy:
     resources:
       limits:
         memory: 2Gi
         cpu: 1
   EOF
   ```

2. **Install with Custom Values**
   ```bash
   helm install nodepulse . -n monitoring -f values-production.yaml
   ```

## Configuration

### Prometheus

1. **Alert Rules**
   ```yaml
   # Example alert rule
   groups:
   - name: example
     rules:
     - alert: HighMemoryUsage
       expr: container_memory_usage_bytes > 1e9
       for: 5m
       labels:
         severity: warning
       annotations:
         summary: High memory usage
   ```

2. **Service Discovery**
   ```yaml
   # Example service discovery
   scrape_configs:
     - job_name: 'kubernetes-pods'
       kubernetes_sd_configs:
         - role: pod
   ```

### Grafana

1. **Dashboard Setup**
   ```bash
   # Import example dashboards
   kubectl create configmap grafana-dashboards \
     --from-file=dashboards/ \
     -n monitoring
   ```

2. **Data Sources**
   ```yaml
   # Example datasource configuration
   datasources:
     - name: Prometheus
       type: prometheus
       url: http://prometheus-server.monitoring.svc.cluster.local
       access: proxy
   ```

### Trivy

1. **Scan Configuration**
   ```yaml
   # Example scan configuration
   vulnerabilityScanner:
     enabled: true
     schedule: "0 0 * * *"
     concurrentScanJobsLimit: 5
   ```

2. **Report Retention**
   ```yaml
   # Example retention policy
   reportRetention:
     enabled: true
     maxAge: 30d
   ```

## Monitoring Setup

### Docker Desktop

1. **Port Forwarding**
   ```bash
   # Grafana
   kubectl port-forward svc/grafana 3000:80 -n monitoring &
   
   # Prometheus
   kubectl port-forward svc/prometheus-server 9090:80 -n monitoring &
   
   # AlertManager
   kubectl port-forward svc/prometheus-alertmanager 9093:80 -n monitoring &
   ```

2. **Access URLs**
   - Grafana: http://localhost:3000
   - Prometheus: http://localhost:9090
   - AlertManager: http://localhost:9093

### Production

1. **Ingress Setup**
   ```yaml
   # Example ingress configuration
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: monitoring
     namespace: monitoring
   spec:
     rules:
     - host: monitoring.example.com
       http:
         paths:
         - path: /grafana
           pathType: Prefix
           backend:
             service:
               name: grafana
               port:
                 number: 80
   ```

2. **TLS Configuration**
   ```yaml
   # Example TLS configuration
   tls:
   - hosts:
     - monitoring.example.com
     secretName: monitoring-tls
   ```

## Security Configuration

### Authentication

1. **Grafana SSO**
   ```yaml
   # Example SSO configuration
   grafana:
     env:
       GF_AUTH_GENERIC_OAUTH_ENABLED: "true"
       GF_AUTH_GENERIC_OAUTH_CLIENT_ID: "your-client-id"
       GF_AUTH_GENERIC_OAUTH_CLIENT_SECRET: "your-client-secret"
   ```

2. **RBAC Setup**
   ```yaml
   # Example RBAC configuration
   rbac:
     create: true
     pspEnabled: false
   ```

### Network Security

1. **Network Policies**
   ```yaml
   # Example network policy
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: monitoring
     namespace: monitoring
   spec:
     podSelector: {}
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             name: monitoring
   ```

2. **TLS Configuration**
   ```bash
   # Generate TLS certificates
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout tls.key -out tls.crt -subj "/CN=monitoring.example.com"
   
   # Create TLS secret
   kubectl create secret tls monitoring-tls \
     --key tls.key --cert tls.crt \
     -n monitoring
   ```

## Troubleshooting

### Common Issues

1. **Prometheus OOM**
   ```bash
   # Check memory usage
   kubectl top pods -n monitoring
   
   # Increase memory limits
   helm upgrade nodepulse . -n monitoring \
     --set prometheus.server.resources.limits.memory=4Gi
   ```

2. **Grafana Login Issues**
   ```bash
   # Reset admin password
   kubectl exec -it $(kubectl get pods -n monitoring -l app.kubernetes.io/name=grafana -o jsonpath='{.items[0].metadata.name}') \
     -n monitoring -- grafana-cli admin reset-admin-password newpassword
   ```

3. **Trivy Scan Failures**
   ```bash
   # Check scan logs
   kubectl logs -n monitoring -l app.kubernetes.io/name=trivy-operator
   
   # Adjust resource limits
   helm upgrade nodepulse . -n monitoring \
     --set trivy.resources.limits.memory=2Gi
   ```

### Log Collection

1. **Component Logs**
   ```bash
   # Prometheus logs
   kubectl logs -n monitoring -l app.kubernetes.io/name=prometheus
   
   # Grafana logs
   kubectl logs -n monitoring -l app.kubernetes.io/name=grafana
   
   # Trivy logs
   kubectl logs -n monitoring -l app.kubernetes.io/name=trivy-operator
   ```

2. **Event Logs**
   ```bash
   # Cluster events
   kubectl get events -n monitoring --sort-by='.lastTimestamp'
   ```

## Support

For additional support:
- GitHub Issues: https://github.com/DARK-art108/nodepulse/issues
- Documentation: https://github.com/DARK-art108/nodepulse/docs
- Community: https://github.com/DARK-art108/nodepulse/discussions 