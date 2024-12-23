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
   ```
2.**Create the monitoring Namespace**
```
  kubectl create namespace monitoring
```
3.**Deploy the Monitoring Stack**
```
 kubectl apply -f main.yaml
```

Let's dive into each component:


1. **Core Components**:

- **Sample App (Nginx)**: A test application being monitored
- **Prometheus**: Metrics collector
- **Loki**: Log aggregator
- **Vector**: Log collector/forwarder
- **Grafana**: Visualization platform

2. **Data Flow**:
```
Metrics Flow: Your Apps -> Prometheus -> Grafana
Logs Flow: Your Apps -> Vector -> Loki -> Grafana
```

Let's dive into each component:

1. **Sample App (nginx)**:
```yaml
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 2  # Runs 2 instances
```
- This is your test application running nginx
- Exposed on port 80
- Has a corresponding service for internal access

2. **Prometheus**:
```yaml
data:
  prometheus.yml: |
    scrape_configs:
      - job_name: 'sample-app'
        static_configs:
          - targets: ['sample-app:80']
```
- Scrapes metrics every 15 seconds
- Currently configured to monitor the sample-app
- To add more apps to monitor, add them under `scrape_configs`

3. **Vector**:
```yaml
kind: DaemonSet  # Runs on every node
data:
  config.yaml: |
    sources:
      kubernetes_logs:  # Collects all container logs
    sinks:
      loki_sink:  # Sends to Loki
```
- Runs on every node (DaemonSet)
- Collects logs from all containers
- Forwards logs to Loki
- Has RBAC permissions to read pod logs

4. **Loki**:
```yaml
data:
  loki.yaml: |
    storage_config:
      boltdb:
        directory: /tmp/loki/index
      filesystem:
        directory: /tmp/loki/chunks
```
- Stores and indexes logs
- Uses local storage (good for testing)
- For production, consider using object storage

5. **Grafana**:
- Frontend for visualization
- Accessible via NodePort

**How to Use This Stack**:

1. **Access Grafana**:
```bash
# Get Grafana URL
kubectl get svc -n monitoring grafana
```
- Default credentials: admin/admin

2. **Configure Data Sources in Grafana**:
   - Add Prometheus:
     - URL: http://prometheus:9090
     - Type: Prometheus
   - Add Loki:
     - URL: http://loki:3100
     - Type: Loki

3. **Create Dashboards**:
   - For Metrics (Prometheus):
   ```
   # Example PromQL query
   rate(http_requests_total{job="sample-app"}[5m])
   ```

   - For Logs (Loki):
   ```
   # Example LogQL query
   {namespace="monitoring"}
   ```

4. **Add More Applications**:

   a. For Metrics:
   ```yaml
   # In prometheus-configmap.yaml
   scrape_configs:
     - job_name: 'new-app'
       static_configs:
         - targets: ['new-app-service:port']
   ```

   b. Logs are automatically collected by Vector

5. **Monitor Resources**:
```bash
# Check pod status
kubectl get pods -n monitoring

# Check logs
kubectl logs -f -n monitoring -l app=vector
kubectl logs -f -n monitoring -l app=loki
kubectl logs -f -n monitoring -l app=prometheus
```

6. **Best Practices**:

- For Production:
  ```yaml
  # Add resource limits
  resources:
    limits:
      memory: "512Mi"
      cpu: "500m"
    requests:
      memory: "256Mi"
      cpu: "250m"
  ```
  
- Add persistent storage:
  ```yaml
  # For Loki
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: loki-pvc
  ```

- Configure retention:
  ```yaml
  # In Loki config
  limits_config:
    retention_period: 30d
  ```

This stack gives you:
- Metrics monitoring (Prometheus)
- Log aggregation (Vector + Loki)
- Visualization (Grafana)
- Auto-discovery of Kubernetes pods
- Centralized logging


