# EFK/ELK Stack — Centralized Logging 📋

Collect, store, and search logs from all your containers in one place.

---

## The Problem

```
Without centralized logging:
  kubectl logs pod-1 -n production
  kubectl logs pod-2 -n production
  kubectl logs pod-3 -n production
  ... 50 more pods ...
  Pod crashed? Logs are GONE. 💀

With EFK Stack:
  All logs → Fluentd/Fluent Bit → Elasticsearch → Kibana
  Search across ALL pods, ALL namespaces, even deleted pods.
```

---

## EFK vs ELK

```
ELK Stack (original):          EFK Stack (Kubernetes):
  Elasticsearch                   Elasticsearch
  Logstash (heavy, Java)          Fluentd / Fluent Bit (lightweight)
  Kibana                          Kibana

Fluent Bit = ultra-lightweight log forwarder (preferred for K8s)
Fluentd = more powerful, more plugins
```

---

## Architecture

```
┌─── Every Node ──────────────────────────────────────────────┐
│                                                              │
│  Pod A → stdout/stderr ──┐                                  │
│  Pod B → stdout/stderr ──┼──→ /var/log/containers/*.log     │
│  Pod C → stdout/stderr ──┘           │                      │
│                                       │                      │
│                            ┌──────────▼──────────┐          │
│                            │  Fluent Bit          │          │
│                            │  (DaemonSet — runs   │          │
│                            │   on every node)     │          │
│                            └──────────┬───────────┘          │
└───────────────────────────────────────┼──────────────────────┘
                                        │
                                        ▼
                            ┌───────────────────────┐
                            │   Elasticsearch        │
                            │   (stores & indexes    │
                            │    all logs)            │
                            └───────────┬────────────┘
                                        │
                                        ▼
                            ┌───────────────────────┐
                            │   Kibana               │
                            │   (search, visualize,  │
                            │    dashboards)          │
                            └────────────────────────┘
```

---

## Install on EKS

### Option 1: Elasticsearch + Fluent Bit + Kibana
```bash
# Elasticsearch
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch \
    --namespace logging \
    --create-namespace \
    --set replicas=2 \
    --set resources.requests.memory=2Gi

# Kibana
helm install kibana elastic/kibana \
    --namespace logging \
    --set service.type=LoadBalancer

# Fluent Bit (DaemonSet)
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit \
    --namespace logging \
    --set output.elasticsearch.host=elasticsearch-master
```

### Option 2: AWS OpenSearch (Managed)
```
AWS manages Elasticsearch for you.
Fluent Bit → OpenSearch → OpenSearch Dashboards (Kibana fork)
Less maintenance, more cost.
```

### Option 3: Loki + Grafana (Lightweight Alternative)
```bash
# Loki = like Prometheus but for logs. Doesn't index full text.
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack \
    --namespace logging \
    --create-namespace \
    --set grafana.enabled=true \
    --set promtail.enabled=true
```

Loki is simpler and cheaper than Elasticsearch. Good for smaller setups.

---

## Fluent Bit Configuration

```yaml
# fluent-bit-config (simplified)
[INPUT]
    Name              tail
    Path              /var/log/containers/*.log
    Parser            docker
    Tag               kube.*
    Refresh_Interval  5

[FILTER]
    Name                kubernetes
    Match               kube.*
    Kube_URL            https://kubernetes.default.svc
    Merge_Log           On
    K8S-Logging.Parser  On

[OUTPUT]
    Name            es
    Match           *
    Host            elasticsearch-master
    Port            9200
    Index           k8s-logs
    Type            _doc
```

What this does:
1. **INPUT** — Reads container log files from every node
2. **FILTER** — Enriches logs with Kubernetes metadata (pod name, namespace, labels)
3. **OUTPUT** — Sends to Elasticsearch

---

## Kibana — Searching Logs

### Common Queries
```
# Find errors in production namespace
kubernetes.namespace_name: "production" AND level: "error"

# Find logs from specific pod
kubernetes.pod_name: "my-app-7d8f9b6c4-abc12"

# Find logs containing specific text
message: "connection refused"

# Find 5xx errors in last 1 hour
status: (500 OR 502 OR 503) AND @timestamp > now-1h

# Find OOMKilled events
message: "OOMKilled" OR message: "Out of memory"
```

### Useful Kibana Features
- **Discover** — Search and filter logs
- **Dashboards** — Visualize log patterns
- **Alerts** — Trigger on log patterns (e.g., error spike)
- **Index patterns** — Define which data to search

---

## EFK vs Loki vs CloudWatch Logs

| Feature | EFK (Elasticsearch) | Loki | CloudWatch Logs |
|---------|-------------------|------|-----------------|
| Full-text search | ✅ Powerful | ❌ Label-based only | ✅ Basic |
| Cost | High (needs lots of RAM) | Low | Pay per GB |
| Maintenance | High | Low | None (managed) |
| K8s native | ✅ | ✅ | Needs agent |
| Grafana integration | Via plugin | ✅ Native | Via plugin |
| Best for | Large scale, complex queries | Small-medium, cost-conscious | AWS-native |

**Recommendation:**
- Small team / budget → **Loki + Grafana**
- Large scale / complex search → **Elasticsearch + Kibana**
- AWS-native / simple → **CloudWatch Logs**

---

*Centralized logging = you can debug anything, even after pods are gone! 📋*
