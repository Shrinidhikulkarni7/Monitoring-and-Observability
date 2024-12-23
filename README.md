# Monitoring-and-Observability

# Kubernetes Monitoring Stack with NGINX, Prometheus, Loki, and Grafana

This repository contains Kubernetes manifests for setting up a monitoring and observability stack using NGINX, Prometheus, Loki, Grafana, and Vector. The stack enables metrics collection and visualization as well as centralized logging for containerized applications.

## Architecture

### Data Flow
1. **NGINX** → **NGINX Exporter** → **Prometheus** → **Grafana**

### Metrics Flow
1. **Container Logs** → **Vector** → **Loki** → **Grafana**

## Components

### 1. NGINX and NGINX Exporter
- **NGINX**: Serves HTTP requests and provides `/metrics` endpoint for scraping.
- **NGINX Exporter**: Collects and exposes NGINX metrics for Prometheus.

### 2. Prometheus
- Collects metrics from the NGINX Exporter and other Kubernetes pods annotated for monitoring.
- Configured with a `prometheus.yml` file to scrape metrics from labeled endpoints.

### 3. Loki and Vector
- **Vector**: Aggregates logs from Kubernetes pods and forwards them to Loki.
- **Loki**: Stores and indexes logs for querying via Grafana.

### 4. Grafana
- Visualizes metrics and logs collected by Prometheus and Loki.
- Configurable with dashboards for real-time observability.

## Kubernetes Manifests

### Deployments
- **NGINX Deployment**: Runs NGINX and the NGINX Exporter in a single pod.
- **Prometheus Deployment**: Single-instance deployment for collecting metrics.
- **Loki Deployment**: Stateful deployment for storing logs.
- **Vector DaemonSet**: Runs on all nodes to collect logs.
- **Grafana Deployment**: Single-instance deployment for visualization.

### ConfigMaps
- **nginx-config**: Configures the NGINX server to expose a `/stub_status` endpoint for metrics.
- **prometheus-config**: Defines scrape intervals, paths, and relabeling rules for Prometheus.
- **vector-config**: Configures Vector to forward logs to Loki.
- **loki-config**: Configures Loki for log ingestion and storage.

### Services
- **NGINX Service**: Exposes the HTTP and metrics ports.
- **Prometheus Service**: Exposes the Prometheus metrics endpoint.
- **Loki Service**: Exposes the Loki log ingestion and query endpoint.
- **Grafana Service**: Exposes the Grafana UI (NodePort configured).

### RBAC
- **Vector RBAC**: Provides Vector with permissions to list and watch pods and namespaces.

## Setup Instructions

1. **Clone the Repository**
   ```bash
   git clone https://github.com/your-repo/monitoring-stack.git
   cd monitoring-stack
