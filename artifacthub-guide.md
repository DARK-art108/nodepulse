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
10. [Component-Specific Guides](#component-specific-guides)

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

## Component-Specific Guides

### OpenTelemetry Configuration

#### 1. Basic Setup
```yaml
# values.yaml
opentelemetry-collector:
  enabled: true
  mode: deployment
  config:
    receivers:
      otlp:
        protocols:
          grpc:
          http:
    processors:
      batch:
    exporters:
      logging:
        verbosity: detailed
      prometheus:
        endpoint: "0.0.0.0:8889"
        namespace: "nodepulse"
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [logging]
        metrics:
          receivers: [otlp]
          processors: [batch]
          exporters: [prometheus]
```

#### 2. Instrumenting Applications
```yaml
# Example application deployment with OpenTelemetry
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  template:
    spec:
      containers:
      - name: example-app
        image: example/app:latest
        env:
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://nodepulse-opentelemetry-collector.monitoring:4317"
        - name: OTEL_SERVICE_NAME
          value: "example-app"
```

#### 3. Accessing Traces
```bash
# Port-forward OpenTelemetry Collector
kubectl port-forward -n monitoring svc/nodepulse-opentelemetry-collector 8889:8889

# Access metrics
curl http://localhost:8889/metrics
```

### Trivy Security Scanner

#### 1. Basic Configuration
```yaml
# values.yaml
trivy-operator:
  vulnerabilityScanner:
    schedule: "0 */6 * * *"
    scanJob:
      resources:
        requests:
          memory: 256Mi
          cpu: 100m
        limits:
          memory: 512Mi
          cpu: 200m
```

#### 2. Viewing Security Reports
```bash
# List vulnerability reports
kubectl get vulnerabilityreports -A

# List config audit reports
kubectl get configauditreports -A

# Get detailed report
kubectl get vulnerabilityreports <pod-name> -o yaml
```

#### 3. Custom Scan Configuration
```yaml
# Example custom scan configuration
apiVersion: aquasecurity.github.io/v1alpha1
kind: ConfigAuditReport
metadata:
  name: example-config-audit
spec:
  scanner:
    name: trivy
    version: "0.18.0"
  target:
    kind: Deployment
    name: example-deployment
```

### Metrics Server

#### 1. Configuration
```yaml
# values.yaml
metrics-server:
  enabled: true
  args:
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP
  resources:
    requests:
      memory: 100Mi
      cpu: 100m
    limits:
      memory: 200Mi
      cpu: 200m
```

#### 2. Verifying Metrics
```bash
# Check metrics server status
kubectl get apiservice v1beta1.metrics.k8s.io

# View node metrics
kubectl top nodes

# View pod metrics
kubectl top pods -A
```

### Loki Log Aggregation

#### 1. Advanced Configuration
```yaml
# values.yaml
loki:
  persistence:
    size: 100Gi
  config:
    auth_enabled: false
    ingester:
      chunk_idle_period: 3m
      chunk_retain_period: 1m
    schema_config:
      configs:
        - from: 2020-10-24
          store: boltdb-shipper
          object_store: filesystem
          schema: v11
          index:
            prefix: index_
            period: 24h
```

#### 2. Querying Logs
```bash
# Using LogCLI
logcli query '{namespace="monitoring"}'

# Using Grafana Explore
# 1. Access Grafana
# 2. Go to Explore
# 3. Select Loki as data source
# 4. Use LogQL queries:
#    - {namespace="monitoring"}
#    - {app="nodepulse"} |= "error"
```

#### 3. Log Retention
```yaml
# Configure log retention
loki:
  config:
    limits_config:
      retention_period: 720h  # 30 days
      max_global_streams_per_user: 5000
```

### Prometheus Configuration

#### 1. Advanced Settings
```yaml
# values.yaml
prometheus:
  server:
    retention: 15d
    retentionSize: "50GiB"
    config:
      global:
        scrape_interval: 15s
        evaluation_interval: 15s
      rule_files:
        - /etc/prometheus/rules/*.yaml
    extraArgs:
      web.enable-lifecycle: "true"
```

#### 2. Custom Rules
```yaml
# Example custom rules
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-rules
  namespace: monitoring
spec:
  groups:
  - name: custom
    rules:
    - alert: HighMemoryUsage
      expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High memory usage detected
```

### Grafana Configuration

#### 1. Dashboard Management
```yaml
# values.yaml
grafana:
  dashboardProviders:
    dashboardproviders.yaml:
      apiVersion: 1
      providers:
      - name: default
        orgId: 1
        folder: ''
        type: file
        disableDeletion: false
        editable: true
        options:
          path: /var/lib/grafana/dashboards
  dashboards:
    default:
      node-exporter:
        gnetId: 1860
        revision: 21
        datasource: Prometheus
```

#### 2. Authentication
```yaml
# values.yaml
grafana:
  adminPassword: your-secure-password
  security:
    adminUser: admin
    adminPassword: your-secure-password
  auth:
    disableLoginForm: false
    disableSignoutMenu: false
    oauth:
      enabled: true
      allowSignUp: true
      autoLogin: false
```

#### 3. Data Sources
```yaml
# values.yaml
grafana:
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
      - name: Prometheus
        type: prometheus
        url: http://nodepulse-prometheus-server
        access: proxy
        isDefault: true
      - name: Loki
        type: loki
        url: http://nodepulse-loki:3100
        access: proxy
```

### OpenTelemetry Observability

#### 1. Metrics Collection
OpenTelemetry collects various types of metrics:

##### Application Metrics
- Request rates and latencies
- Error rates and types
- Resource utilization (CPU, memory)
- Custom business metrics
- Database query performance
- Cache hit/miss ratios

##### System Metrics
- Container metrics
- Process metrics
- Network metrics
- Disk I/O metrics
- System calls

##### Kubernetes Metrics
- Pod lifecycle events
- Container resource usage
- Node resource utilization
- Network traffic
- Storage usage

#### 2. Distributed Tracing
Track requests across services:

##### Trace Components
- Service maps showing dependencies
- Request flow visualization
- Latency breakdowns
- Error propagation
- Context propagation

##### Trace Attributes
- Service names
- Operation names
- Request IDs
- User IDs
- Custom attributes

#### 3. Log Collection
Unified log management:

##### Log Types
- Application logs
- System logs
- Audit logs
- Security logs
- Performance logs

##### Log Attributes
- Timestamps
- Log levels
- Source information
- Context data
- Correlation IDs

#### 4. Visualization in Grafana

##### Pre-configured Dashboards
1. **Service Overview Dashboard**
   - Request rates
   - Error rates
   - Latency percentiles
   - Resource usage
   - Dependencies map

2. **Trace Explorer**
   - Service map
   - Trace timeline
   - Span details
   - Error analysis
   - Performance bottlenecks

3. **Resource Monitoring**
   - CPU usage
   - Memory usage
   - Network traffic
   - Disk I/O
   - System metrics

4. **Application Performance**
   - Response times
   - Throughput
   - Error rates
   - Cache performance
   - Database performance

#### 5. Example Queries

##### Metrics Queries
```promql
# Request rate
rate(http_server_duration_seconds_count[5m])

# Error rate
rate(http_server_duration_seconds_count{status_code=~"5.."}[5m])

# Latency
histogram_quantile(0.95, sum(rate(http_server_duration_seconds_bucket[5m])) by (le))

# Resource usage
rate(process_cpu_seconds_total[5m])
process_resident_memory_bytes
```

##### Log Queries
```logql
# Error logs
{service="my-service"} |= "error"

# High latency requests
{service="my-service"} | json | latency > 1000

# Specific user activity
{service="my-service"} | json | user_id="12345"
```

##### Trace Queries
```jaeger
# Find slow requests
service="my-service" AND duration>1s

# Find errors
service="my-service" AND error=true

# Find specific operation
service="my-service" AND operation="process_order"
```

#### 6. Alerting Examples

```yaml
# High latency alert
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: high-latency
  namespace: monitoring
spec:
  groups:
  - name: latency
    rules:
    - alert: HighRequestLatency
      expr: histogram_quantile(0.95, sum(rate(http_server_duration_seconds_bucket[5m])) by (le)) > 1
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High request latency detected
        description: 95th percentile latency is above 1 second

# Error rate alert
    - alert: HighErrorRate
      expr: rate(http_server_duration_seconds_count{status_code=~"5.."}[5m]) / rate(http_server_duration_seconds_count[5m]) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: High error rate detected
        description: Error rate is above 5%
```

#### 7. Best Practices

1. **Instrumentation**
   - Use auto-instrumentation where possible
   - Add custom spans for business logic
   - Include relevant context in traces
   - Use consistent naming conventions

2. **Sampling**
   - Sample traces based on volume
   - Keep all error traces
   - Adjust sampling rates based on importance
   - Use dynamic sampling for high-volume services

3. **Context Propagation**
   - Propagate trace context across services
   - Include user context where relevant
   - Add business context to traces
   - Maintain correlation IDs

4. **Resource Management**
   - Set appropriate retention periods
   - Configure sampling strategies
   - Monitor collector resource usage
   - Use batching for efficiency

### OpenTelemetry Exploration Guide

#### 1. Access OpenTelemetry Collector

```bash
# Port-forward OpenTelemetry Collector
kubectl port-forward -n monitoring svc/nodepulse-opentelemetry-collector 8889:8889

# Check collector health
curl http://localhost:8889/healthz

# View collector metrics
curl http://localhost:8889/metrics
```

#### 2. View Traces

```bash
# Port-forward Jaeger UI
kubectl port-forward -n monitoring svc/nodepulse-jaeger-query 16686:16686

# Access Jaeger UI
# Open browser: http://localhost:16686

# Query traces using curl
curl -X GET "http://localhost:16686/api/traces?service=my-service&limit=20"
```

#### 3. Explore Metrics

```bash
# Port-forward Prometheus
kubectl port-forward -n monitoring svc/nodepulse-prometheus-server 9090:9090

# Query OpenTelemetry metrics
curl -G "http://localhost:9090/api/v1/query" \
  --data-urlencode "query=rate(otel_http_server_duration_seconds_count[5m])"

# View service metrics
curl -G "http://localhost:9090/api/v1/query" \
  --data-urlencode "query=otel_service_instance_count"
```

#### 4. Check Logs

```bash
# Port-forward Loki
kubectl port-forward -n monitoring svc/nodepulse-loki 3100:3100

# Query logs using LogCLI
logcli query '{service="my-service"}'

# View specific log entries
logcli query '{service="my-service"} |= "error"'
```

#### 5. Service Map Exploration

```bash
# View service dependencies
curl -G "http://localhost:16686/api/dependencies" \
  --data-urlencode "endTs=$(date +%s)000000" \
  --data-urlencode "lookback=3600000"

# Get service operations
curl -G "http://localhost:16686/api/services/my-service/operations"
```

#### 6. Trace Analysis

```bash
# Get trace by ID
curl -G "http://localhost:16686/api/traces/{trace-id}"

# Search traces by tags
curl -G "http://localhost:16686/api/traces" \
  --data-urlencode "service=my-service" \
  --data-urlencode "tag=error=true"
```

#### 7. Metrics Analysis

```bash
# Get service metrics
curl -G "http://localhost:9090/api/v1/query" \
  --data-urlencode "query=rate(otel_http_server_duration_seconds_sum[5m]) / rate(otel_http_server_duration_seconds_count[5m])"

# Check error rates
curl -G "http://localhost:9090/api/v1/query" \
  --data-urlencode "query=rate(otel_http_server_duration_seconds_count{status_code=~'5..'}[5m])"
```

#### 8. Resource Usage

```bash
# Check collector resource usage
kubectl top pod -n monitoring -l app.kubernetes.io/name=opentelemetry-collector

# View collector logs
kubectl logs -n monitoring -l app.kubernetes.io/name=opentelemetry-collector
```

#### 9. Export Data

```bash
# Export traces (requires jq)
curl -s "http://localhost:16686/api/traces?service=my-service" | jq .

# Export metrics
curl -s "http://localhost:9090/api/v1/query_range" \
  --data-urlencode "query=rate(otel_http_server_duration_seconds_count[5m])" \
  --data-urlencode "start=$(date -d '1 hour ago' +%s)" \
  --data-urlencode "end=$(date +%s)" \
  --data-urlencode "step=15" | jq .
```

#### 10. Debugging

```bash
# Check collector configuration
kubectl get configmap -n monitoring nodepulse-opentelemetry-collector -o yaml

# View collector status
kubectl describe pod -n monitoring -l app.kubernetes.io/name=opentelemetry-collector

# Check receiver status
curl -s http://localhost:8889/metrics | grep otelcol_receiver
```

#### 11. Common Queries

```bash
# Find slow requests
curl -G "http://localhost:16686/api/traces" \
  --data-urlencode "service=my-service" \
  --data-urlencode "minDuration=1s"

# Find errors
curl -G "http://localhost:16686/api/traces" \
  --data-urlencode "service=my-service" \
  --data-urlencode "tag=error=true"

# Get service latency
curl -G "http://localhost:9090/api/v1/query" \
  --data-urlencode "query=histogram_quantile(0.95, sum(rate(otel_http_server_duration_seconds_bucket[5m])) by (le))"
``` 