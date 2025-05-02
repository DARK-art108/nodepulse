# NodePulse - Local Setup Guide

This guide will walk you through setting up NodePulse in a local Kubernetes environment and accessing all its dashboards.

## Prerequisites

1. **Local Kubernetes Cluster**
   - [Minikube](https://minikube.sigs.k8s.io/docs/start/) or
   - [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) or
   - [Docker Desktop with Kubernetes](https://docs.docker.com/desktop/kubernetes/)

2. **Required Tools**
   ```bash
   # Install kubectl
   brew install kubectl  # macOS
   # or
   sudo apt-get install kubectl  # Ubuntu

   # Install Helm
   brew install helm  # macOS
   # or
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash  # Linux
   ```

## Local Cluster Setup

### Option 1: Using Minikube

```bash
# Start Minikube with sufficient resources
minikube start --memory=8192 --cpus=4 --disk-size=50g

# Enable required addons
minikube addons enable metrics-server
minikube addons enable default-storageclass
minikube addons enable storage-provisioner

# Verify cluster status
kubectl get nodes
```

### Option 2: Using Kind

```bash
# Create Kind cluster configuration
cat > kind-config.yaml << EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
  - containerPort: 30001
    hostPort: 30001
  - containerPort: 30002
    hostPort: 30002
- role: worker
- role: worker
EOF

# Create the cluster
kind create cluster --config kind-config.yaml

# Verify cluster status
kubectl get nodes
```

## Installing NodePulse

1. **Add Required Helm Repositories**
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo add grafana https://grafana.github.io/helm-charts
   helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server
   helm repo add aquasecurity https://aquasecurity.github.io/helm-charts
   helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
   helm repo update
   ```

2. **Create Monitoring Namespace**
   ```bash
   kubectl create namespace monitoring
   ```

3. **Install NodePulse**
   ```bash
   # Navigate to the chart directory
   cd monitoring-stack

   # Install with custom values for local setup
   helm install nodepulse . -n monitoring \
     --set prometheus.server.retention=7d \
     --set grafana.adminPassword=admin \
     --set loki.persistence.size=20Gi \
     --set prometheus.server.resources.requests.memory=1Gi \
     --set prometheus.server.resources.requests.cpu=500m
   ```

4. **Verify Installation**
   ```bash
   # Check all pods are running
   kubectl get pods -n monitoring -w

   # Check services
   kubectl get svc -n monitoring
   ```

## Accessing Dashboards

### 1. Grafana Dashboard

```bash
# Port-forward Grafana service
kubectl port-forward -n monitoring svc/nodepulse-grafana 3000:80
```

Access Grafana:
- URL: http://localhost:3000
- Username: admin
- Password: admin

Pre-configured Dashboards:
- Kubernetes / Compute Resources / Cluster
- Kubernetes / Compute Resources / Namespace (Pods)
- Kubernetes / Compute Resources / Pod
- Kubernetes / Compute Resources / Workload

### 2. Prometheus UI

```bash
# Port-forward Prometheus service
kubectl port-forward -n monitoring svc/nodepulse-prometheus-server 9090:9090
```

Access Prometheus:
- URL: http://localhost:9090
- No authentication required locally

### 3. Loki (Logs)

```bash
# Port-forward Loki service
kubectl port-forward -n monitoring svc/nodepulse-loki 3100:3100
```

Access Loki:
- URL: http://localhost:3100
- No authentication required locally

### 4. Security Dashboard (Trivy)

```bash
# Check vulnerability reports
kubectl get vulnerabilityreports -A

# Check config audit reports
kubectl get configauditreports -A
```

## Local Development Tips

### 1. Resource Monitoring

```bash
# Monitor cluster resources
kubectl top nodes
kubectl top pods -n monitoring

# Check resource usage
kubectl describe nodes
```

### 2. Log Access

```bash
# View component logs
kubectl logs -n monitoring -l app.kubernetes.io/name=nodepulse

# Follow logs
kubectl logs -n monitoring -f deployment/nodepulse-prometheus-server
```

### 3. Storage Management

```bash
# Check persistent volumes
kubectl get pvc -n monitoring

# Check storage usage
kubectl exec -n monitoring -it <prometheus-pod> -- df -h
```

### 4. Network Access

```bash
# Get service URLs
minikube service list  # For Minikube
# or
kubectl get svc -n monitoring

# Test service connectivity
curl http://localhost:3000/api/health  # Grafana
curl http://localhost:9090/-/healthy   # Prometheus
```

## Troubleshooting Local Setup

### 1. Pod Not Starting

```bash
# Check pod events
kubectl describe pod -n monitoring <pod-name>

# Check pod logs
kubectl logs -n monitoring <pod-name>
```

### 2. Storage Issues

```bash
# Check PVC status
kubectl get pvc -n monitoring

# Check storage class
kubectl get storageclass

# If using Minikube, check storage provisioner
minikube addons list | grep storage
```

### 3. Network Issues

```bash
# Check service endpoints
kubectl get endpoints -n monitoring

# Test service connectivity
kubectl run -it --rm --restart=Never --image=curlimages/curl curl -- \
  curl http://nodepulse-prometheus-server.monitoring:9090/-/healthy
```

### 4. Resource Issues

```bash
# Check resource limits
kubectl describe pod -n monitoring <pod-name>

# Adjust resources if needed
helm upgrade nodepulse . -n monitoring \
  --set prometheus.server.resources.requests.memory=2Gi \
  --set prometheus.server.resources.requests.cpu=1000m
```

## Cleanup

```bash
# Uninstall NodePulse
helm uninstall nodepulse -n monitoring

# Delete namespace
kubectl delete namespace monitoring

# If using Minikube
minikube stop
minikube delete

# If using Kind
kind delete cluster
```

## Next Steps

1. Configure persistent storage for production
2. Set up proper authentication
3. Configure custom dashboards
4. Set up alerting rules
5. Configure backup and restore procedures

For production setup, refer to the main [README.md](README.md) for security best practices and advanced configuration options. 