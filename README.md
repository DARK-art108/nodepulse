# NodePulse - Kubernetes Monitoring Suite

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/nodepulse)](https://artifacthub.io/packages/helm/nodepulse/nodepulse)

NodePulse is a plug-and-play observability stack for Kubernetes, combining Prometheus, Grafana, Loki, Trivy, OpenTelemetry, and more to deliver unified metrics, logs, traces, and security insights.

## Features

- **Prometheus**: Metrics collection and alerting
- **Grafana**: Visualization and dashboards
- **Loki**: Log aggregation
- **Metrics Server**: Kubernetes metrics API implementation
- **Trivy**: Security scanning
- **OpenTelemetry**: Distributed tracing

## Prerequisites

- Kubernetes cluster (version 1.16 or later)
- Helm 3.x
- kubectl configured to access your cluster

## Installation

1. Add the required Helm repositories:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server
helm repo add aquasecurity https://aquasecurity.github.io/helm-charts
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```

2. Create the monitoring namespace:
```bash
kubectl create namespace monitoring
```

3. Install NodePulse:
```bash
helm install nodepulse . -n monitoring
```

## Configuration

The following table lists the configurable parameters of the NodePulse chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `prometheus.enabled` | Enable Prometheus | `true` |
| `grafana.enabled` | Enable Grafana | `true` |
| `loki.enabled` | Enable Loki | `true` |
| `metrics-server.enabled` | Enable Metrics Server | `true` |
| `trivy-operator.enabled` | Enable Trivy Operator | `true` |
| `opentelemetry-collector.enabled` | Enable OpenTelemetry Collector | `true` |

For more configuration options, please refer to the [values.yaml](values.yaml) file.

## Accessing the Components

- **Grafana**: `http://<grafana-service-ip>:3000`
  - Default credentials: admin/admin (change in production)
- **Prometheus**: `http://<prometheus-service-ip>:9090`
- **Loki**: `http://<loki-service-ip>:3100`

## Updating NodePulse

To update NodePulse with new values:

```bash
helm upgrade nodepulse . -n monitoring
```

## Uninstalling

To uninstall NodePulse:

```bash
helm uninstall nodepulse -n monitoring
kubectl delete namespace monitoring
```

## Security Considerations

1. Change the default Grafana admin password in production
2. Configure proper TLS for all components
3. Set up proper RBAC and network policies
4. Configure proper resource limits based on your cluster size

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Monitoring and Alerts

NodePulse comes with basic monitoring and alerting configured. You can customize these by:

1. Modifying Prometheus alert rules
2. Creating custom Grafana dashboards
3. Configuring alert notifications in Grafana
4. Setting up log-based alerts in Loki

## Troubleshooting

Common issues and solutions:

1. **Storage Issues**: Ensure your cluster has proper storage classes configured
2. **Resource Constraints**: Adjust resource limits in values.yaml if components are failing
3. **Network Issues**: Check network policies and service configurations
4. **Authentication Issues**: Verify RBAC configurations and service account permissions 