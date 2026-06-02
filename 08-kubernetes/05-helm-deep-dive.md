# Helm — Deep Dive 📦

The package manager for Kubernetes, explained thoroughly.

---

## Why Helm?

```
Without Helm:                          With Helm:
├── deployment.yaml                    helm install my-app ./chart \
├── service.yaml                         --set env=prod \
├── configmap.yaml                       --set replicas=3 \
├── secret.yaml                          --set image.tag=v2.0
├── ingress.yaml
├── hpa.yaml                           ONE command. All files. Templated.
├── pdb.yaml
└── serviceaccount.yaml

8 files × 3 environments = 24 files   1 chart × 3 value files = done
```

---

## Chart Structure (In Detail)

```
my-app/
├── Chart.yaml              ← Metadata: name, version, description
├── values.yaml             ← Default values (overridden per environment)
├── values-dev.yaml         ← Dev-specific overrides
├── values-staging.yaml     ← Staging-specific overrides
├── values-prod.yaml        ← Prod-specific overrides
├── templates/
│   ├── _helpers.tpl        ← Reusable template snippets
│   ├── deployment.yaml     ← Templated Kubernetes manifests
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   ├── serviceaccount.yaml
│   └── NOTES.txt           ← Post-install message
└── charts/                 ← Sub-chart dependencies
```

### Chart.yaml
```yaml
apiVersion: v2
name: my-app
description: My application Helm chart
type: application
version: 1.0.0        # Chart version
appVersion: "2.0.0"   # Application version
dependencies:
  - name: redis
    version: "17.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

### values.yaml (Defaults)
```yaml
replicaCount: 2

image:
  repository: 123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: alb
  host: myapp.example.com
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilization: 70

env:
  LOG_LEVEL: info
  DB_HOST: ""

secrets:
  DB_PASSWORD: ""

redis:
  enabled: false
```

### values-prod.yaml (Production Overrides)
```yaml
replicaCount: 3

image:
  tag: "v2.0.0"

resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi

autoscaling:
  minReplicas: 3
  maxReplicas: 20

env:
  LOG_LEVEL: warn
  DB_HOST: prod-db.cluster-abc.ca-central-1.rds.amazonaws.com

redis:
  enabled: true
```

### Templated deployment.yaml
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 8080
          env:
            {{- range $key, $value := .Values.env }}
            - name: {{ $key }}
              value: {{ $value | quote }}
            {{- end }}
          envFrom:
            - secretRef:
                name: {{ include "my-app.fullname" . }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
```

---

## Helm Commands — Complete Reference

```bash
# ─── Repository Management ───
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm repo list
helm search repo nginx
helm search hub wordpress          # Search Artifact Hub

# ─── Install / Upgrade ───
helm install my-app ./chart                          # From local chart
helm install my-app bitnami/nginx                    # From repo
helm install my-app ./chart -f values-prod.yaml      # Custom values file
helm install my-app ./chart --set replicas=3         # Override single value
helm install my-app ./chart -n production            # Specific namespace
helm install my-app ./chart --create-namespace       # Create ns if missing

helm upgrade my-app ./chart -f values-prod.yaml      # Upgrade existing
helm upgrade --install my-app ./chart                 # Install or upgrade

# ─── Inspect ───
helm list                          # List releases
helm list -A                       # All namespaces
helm status my-app                 # Release status
helm get values my-app             # Current values
helm get manifest my-app           # Rendered manifests
helm history my-app                # Revision history

# ─── Rollback ───
helm rollback my-app 1             # Rollback to revision 1
helm rollback my-app 0             # Rollback to previous

# ─── Debug ───
helm template my-app ./chart                         # Render locally (no install)
helm template my-app ./chart -f values-prod.yaml     # With values
helm install my-app ./chart --dry-run --debug        # Dry run against cluster
helm lint ./chart                                     # Check for issues

# ─── Cleanup ───
helm uninstall my-app
helm uninstall my-app --keep-history                 # Keep history for rollback

# ─── Package & Share ───
helm package ./chart                                 # Create .tgz
helm push my-app-1.0.0.tgz oci://my-registry        # Push to OCI registry
```

---

## Helm + ArgoCD

ArgoCD can deploy Helm charts directly:

```yaml
# ArgoCD Application using Helm
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/myorg/k8s-manifests.git
    path: charts/my-app
    helm:
      valueFiles:
        - values-prod.yaml
      parameters:
        - name: image.tag
          value: "abc123"
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

---

*Helm = write once, deploy everywhere with different values! 📦*
