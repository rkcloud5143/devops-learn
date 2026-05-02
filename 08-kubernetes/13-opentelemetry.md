# OpenTelemetry — Observability Standard 🔭

The emerging standard for metrics, logs, and traces across all platforms.

---

## What is OpenTelemetry (OTel)?

```
Before OTel:
  Datadog SDK for metrics    → locked to Datadog
  Jaeger SDK for traces      → locked to Jaeger
  Fluentd for logs           → separate config
  Switch vendors? Rewrite everything.

With OTel:
  ONE SDK for metrics, traces, and logs
  Send to ANY backend (Datadog, Prometheus, Jaeger, Grafana, AWS X-Ray)
  Switch vendors? Change the exporter config. Zero code changes.

┌─── Your App ──────────┐
│  OTel SDK              │
│  (metrics + traces +   │──→ OTel Collector ──→ Datadog
│   logs)                │                   ──→ Prometheus
└────────────────────────┘                   ──→ Jaeger
                                             ──→ AWS X-Ray
                                             ──→ Grafana Cloud
```

---

## Core Components

```
Signals (what you collect):
  📊 Metrics  — Counters, gauges, histograms (CPU, request count, latency)
  🔗 Traces   — Request flow across services (distributed tracing)
  📋 Logs     — Text records (errors, events)

Components:
  SDK         — Instrument your code (auto or manual)
  Collector   — Receives, processes, exports telemetry data
  Exporters   — Send data to backends (Prometheus, Jaeger, Datadog)
```

---

## OTel Collector on Kubernetes

```yaml
# Deploy as DaemonSet (one per node) or Deployment
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel
spec:
  mode: daemonset
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318

    processors:
      batch:
        timeout: 5s

    exporters:
      prometheus:
        endpoint: 0.0.0.0:8889
      otlp/jaeger:
        endpoint: jaeger-collector:4317
        tls:
          insecure: true

    service:
      pipelines:
        metrics:
          receivers: [otlp]
          processors: [batch]
          exporters: [prometheus]
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlp/jaeger]
```

---

## Auto-Instrumentation (Zero Code Changes)

```yaml
# For Java, Python, Node.js — OTel can auto-instrument
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: my-instrumentation
spec:
  exporter:
    endpoint: http://otel-collector:4317
  propagators:
    - tracecontext
    - baggage
  sampler:
    type: parentbased_traceidratio
    argument: "0.25"    # Sample 25% of traces

# Annotate your deployment:
#   instrumentation.opentelemetry.io/inject-python: "true"
#   instrumentation.opentelemetry.io/inject-nodejs: "true"
#   instrumentation.opentelemetry.io/inject-java: "true"
```

---

## OTel vs Vendor SDKs

| Feature | Vendor SDK (Datadog, etc.) | OpenTelemetry |
|---------|---------------------------|---------------|
| Vendor lock-in | Yes | No |
| Setup | Easy | More config |
| Features | Vendor-specific extras | Standard |
| Switch backends | Rewrite code | Change config |
| Community | Vendor-driven | CNCF open-source |
| Maturity | Mature | Traces mature, metrics stable, logs emerging |

**Recommendation:** Use OTel for new projects. It's the future.

---

*OTel = instrument once, send anywhere. The future of observability! 🔭*
