# Service Mesh — Istio & Linkerd 🕸️

Advanced networking for microservices on Kubernetes.

---

## The Problem

```
Without service mesh:
  App A ──→ App B ──→ App C
  
  Who handles:
  ❓ Retries when B fails?          → Your code
  ❓ Timeout if B is slow?          → Your code
  ❓ Encryption between services?   → Your code
  ❓ Traffic splitting (canary)?    → Manual
  ❓ Observability (which call is slow)? → Instrument everything
  
  Every team implements this differently. Bugs everywhere.

With service mesh:
  App A ──[proxy]──→ [proxy]── App B ──[proxy]──→ [proxy]── App C
  
  The proxy (sidecar) handles ALL of the above.
  Your code just makes HTTP calls. The mesh handles the rest.
```

---

## How It Works

```
┌─── Pod ──────────────────────┐
│                               │
│  ┌─── Your App ────────┐    │
│  │  Makes HTTP call     │    │
│  │  to service-b:8080   │    │
│  └──────────┬───────────┘    │
│             │                 │
│             ▼                 │
│  ┌─── Sidecar Proxy ───┐    │    Sidecar is automatically injected.
│  │  (Envoy)             │    │    Your app doesn't know it's there.
│  │  - mTLS encryption   │    │
│  │  - Retries           │    │
│  │  - Timeouts          │    │
│  │  - Circuit breaking  │    │
│  │  - Metrics           │    │
│  └──────────┬───────────┘    │
└─────────────┼────────────────┘
              │ (encrypted)
              ▼
┌─── Pod (service-b) ─────────┐
│  ┌─── Sidecar Proxy ───┐    │
│  │  Decrypts, forwards  │    │
│  └──────────┬───────────┘    │
│             ▼                 │
│  ┌─── Your App ────────┐    │
│  │  Receives request    │    │
│  └──────────────────────┘    │
└──────────────────────────────┘
```

---

## What a Service Mesh Gives You

| Feature | Without Mesh | With Mesh |
|---------|-------------|-----------|
| **mTLS** (encryption) | Manual cert management | Automatic, zero-config |
| **Retries** | Code it yourself | Config in YAML |
| **Timeouts** | Code it yourself | Config in YAML |
| **Circuit breaking** | Library (Hystrix) | Config in YAML |
| **Traffic splitting** | Manual | 90% v1, 10% v2 in YAML |
| **Observability** | Instrument every service | Automatic metrics, traces |
| **Access control** | Network policies | Fine-grained (service-to-service) |

---

## Istio

The most feature-rich service mesh. More complex.

### Install
```bash
# Install istioctl
brew install istioctl

# Install Istio on cluster
istioctl install --set profile=demo

# Enable sidecar injection for namespace
kubectl label namespace production istio-injection=enabled

# All new pods in "production" namespace get Envoy sidecar automatically
```

### Traffic Management
```yaml
# Canary deployment: 90% to v1, 10% to v2
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app
spec:
  hosts:
    - my-app
  http:
    - route:
        - destination:
            host: my-app
            subset: v1
          weight: 90
        - destination:
            host: my-app
            subset: v2
          weight: 10
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

### Circuit Breaking
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app
spec:
  host: my-app
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: DEFAULT
        http1MaxPendingRequests: 100
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
```

### Observability (Built-in)
```bash
# Kiali — Service mesh dashboard
istioctl dashboard kiali

# Jaeger — Distributed tracing
istioctl dashboard jaeger

# Grafana — Metrics
istioctl dashboard grafana
```

---

## Linkerd

Simpler, lighter alternative to Istio.

```bash
# Install
brew install linkerd
linkerd install | kubectl apply -f -
linkerd check

# Inject sidecar into namespace
kubectl annotate namespace production linkerd.io/inject=enabled

# Dashboard
linkerd dashboard
```

---

## Istio vs Linkerd

| Feature | Istio | Linkerd |
|---------|-------|---------|
| Complexity | High | Low |
| Resource usage | Heavy | Light |
| Learning curve | Steep | Gentle |
| Features | Everything | Core features |
| Proxy | Envoy | Linkerd2-proxy (Rust) |
| Traffic management | Advanced | Basic |
| Best for | Large orgs, complex needs | Small-medium, simplicity |

---

## When Do You Need a Service Mesh?

```
✅ You need it if:
  - 10+ microservices communicating
  - Need mTLS between all services
  - Need canary/traffic splitting
  - Need service-to-service observability
  - Compliance requires encrypted internal traffic

❌ You DON'T need it if:
  - < 5 services
  - Monolith or simple architecture
  - Just starting with Kubernetes
  - Team is small and can't maintain it
```

---

*Service mesh = infrastructure-level networking superpowers. Use when you outgrow basic K8s networking! 🕸️*
