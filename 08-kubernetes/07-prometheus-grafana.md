# Prometheus & Grafana — Monitoring for Kubernetes 📊

The industry-standard monitoring stack.

---

## The Big Picture

```
┌─── Your Kubernetes Cluster ──────────────────────────────────────┐
│                                                                   │
│  ┌─── App Pods ──┐  ┌─── Node ──────┐  ┌─── K8s API ──────┐   │
│  │ /metrics       │  │ node-exporter  │  │ kube-state-metrics│   │
│  │ endpoint       │  │ (CPU, mem,     │  │ (pod status,      │   │
│  │                │  │  disk, net)    │  │  deployments)     │   │
│  └───────┬────────┘  └──────┬────────┘  └──────┬────────────┘   │
│          │                   │                   │                │
│          └───────────────────┼───────────────────┘                │
│                              │                                    │
│                              ▼                                    │
│                    ┌─── Prometheus ──────────┐                   │
│                    │  Scrapes /metrics        │                   │
│                    │  Stores time-series data │                   │
│                    │  Evaluates alert rules   │                   │
│                    └──────────┬──────────────┘                   │
│                               │                                   │
│                    ┌──────────┼──────────┐                       │
│                    ▼                     ▼                        │
│          ┌─── Grafana ──┐    ┌─── Alertmanager ──┐              │
│          │ Dashboards    │    │ Routes alerts to   │              │
│          │ Visualizations│    │ Slack, PagerDuty,  │              │
│          │ Queries       │    │ email              │              │
│          └───────────────┘    └────────────────────┘              │
└───────────────────────────────────────────────────────────────────┘
```

---

## Prometheus

### What It Does
- **Scrapes** metrics from targets (pods, nodes, services) via HTTP `/metrics` endpoint
- **Stores** time-series data (metric name + labels + timestamp + value)
- **Queries** with PromQL (Prometheus Query Language)
- **Alerts** when conditions are met

### Install on EKS (kube-prometheus-stack)
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
    --namespace monitoring \
    --create-namespace \
    --set grafana.adminPassword=admin123
```

This installs: Prometheus, Grafana, Alertmanager, node-exporter, kube-state-metrics — everything.

### How Scraping Works
```yaml
# Prometheus discovers targets via ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

Your app exposes metrics at `/metrics`:
```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",path="/api/users",status="200"} 1234
http_requests_total{method="POST",path="/api/users",status="500"} 5

# HELP http_request_duration_seconds Request duration
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.1"} 900
http_request_duration_seconds_bucket{le="0.5"} 1100
http_request_duration_seconds_bucket{le="1.0"} 1200
```

### PromQL (Query Language)

```promql
# Current CPU usage per pod
rate(container_cpu_usage_seconds_total{namespace="production"}[5m])

# Memory usage in MB
container_memory_usage_bytes{namespace="production"} / 1024 / 1024

# Request rate (requests per second)
rate(http_requests_total[5m])

# Error rate (percentage)
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Top 5 pods by CPU
topk(5, rate(container_cpu_usage_seconds_total[5m]))

# Disk usage percentage
(node_filesystem_size_bytes - node_filesystem_avail_bytes) / node_filesystem_size_bytes * 100
```

### Metric Types
| Type | What it is | Example |
|------|-----------|---------|
| **Counter** | Only goes up | Total requests, total errors |
| **Gauge** | Goes up and down | Current CPU, memory, temperature |
| **Histogram** | Distribution of values | Request duration buckets |
| **Summary** | Like histogram with pre-calculated percentiles | Request duration percentiles |

### Alert Rules
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-alerts
  namespace: monitoring
spec:
  groups:
    - name: my-app
      rules:
        - alert: HighErrorRate
          expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "High error rate on {{ $labels.service }}"
            description: "Error rate is {{ $value | humanizePercentage }}"

        - alert: PodCrashLooping
          expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "Pod {{ $labels.pod }} is crash looping"

        - alert: HighMemoryUsage
          expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "Pod {{ $labels.pod }} memory > 90%"
```

---

## Grafana

### What It Does
- **Visualizes** Prometheus data in dashboards
- **Pre-built dashboards** for K8s, nodes, pods (import by ID)
- **Alerting** (can also alert, not just Alertmanager)

### Useful Dashboard IDs (Import in Grafana)
```
Dashboard ID    What it shows
─────────────   ──────────────────────────────
315             Kubernetes cluster overview
6417            Kubernetes pods
1860            Node Exporter (server metrics)
7249            Kubernetes cluster monitoring
12740           Kubernetes pod resources
```

Import: Grafana → + → Import → Enter ID → Select Prometheus data source.

### Access Grafana on EKS
```bash
# Port forward
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring

# Open http://localhost:3000
# Default: admin / admin123 (or whatever you set)
```

---

## Alertmanager

Routes alerts from Prometheus to notification channels.

```yaml
# alertmanager-config.yaml
route:
  receiver: 'slack-critical'
  group_by: ['alertname', 'namespace']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
    - match:
        severity: critical
      receiver: 'slack-critical'
    - match:
        severity: warning
      receiver: 'slack-warning'

receivers:
  - name: 'slack-critical'
    slack_configs:
      - channel: '#alerts-critical'
        api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
        title: '🔴 {{ .GroupLabels.alertname }}'
        text: '{{ .CommonAnnotations.description }}'

  - name: 'slack-warning'
    slack_configs:
      - channel: '#alerts-warning'
        api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
```

---

## Prometheus vs CloudWatch

| Feature | Prometheus + Grafana | CloudWatch |
|---------|---------------------|------------|
| Cost | Free (self-hosted) | Pay per metric/alarm |
| K8s native | ✅ Built for K8s | Needs Container Insights |
| Custom metrics | Easy (expose /metrics) | SDK required |
| Query language | PromQL (powerful) | Limited |
| Dashboards | Grafana (beautiful) | Basic |
| Community | Huge, many exporters | AWS only |
| Maintenance | You manage it | AWS manages it |

**Use Prometheus** for Kubernetes-heavy environments.
**Use CloudWatch** for AWS-native services (Lambda, RDS, etc.).
**Use both** in most real setups.

---

*Prometheus + Grafana = the eyes of your Kubernetes cluster! 📊*
