# Kubernetes — Helm (Package Manager) & EKS Deep Dive

---

## HELM

```
Helm = package manager for Kubernetes (like apt for Linux)

Instead of writing 10+ YAML files, install an app with one command:
  helm install my-nginx ingress-nginx/ingress-nginx

┌─── Without Helm ─────────────────┐  ┌─── With Helm ──────────────────┐
│                                   │  │                                │
│  kubectl apply -f deployment.yml  │  │  helm install my-app ./chart  │
│  kubectl apply -f service.yml     │  │                                │
│  kubectl apply -f configmap.yml   │  │  (one command, all files)     │
│  kubectl apply -f secret.yml      │  │  (templated values)           │
│  kubectl apply -f ingress.yml     │  │  (easy upgrade/rollback)      │
│  kubectl apply -f hpa.yml         │  │                                │
└───────────────────────────────────┘  └────────────────────────────────┘
```

### Helm Chart Structure
```
my-app-chart/
├── Chart.yaml              ← Chart metadata (name, version)
├── values.yaml             ← Default configuration values
├── templates/
│   ├── deployment.yaml     ← Templated K8s manifests
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── hpa.yaml
└── charts/                 ← Dependencies (sub-charts)
```

### Helm Commands
```bash
# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx

# Install
helm install my-release bitnami/nginx           # From repo
helm install my-app ./my-app-chart               # From local chart
helm install my-app ./chart -f prod-values.yaml  # Custom values

# List installed releases
helm list
helm list -n production                          # In namespace

# Upgrade
helm upgrade my-app ./chart -f prod-values.yaml

# Rollback
helm rollback my-app 1                           # Rollback to revision 1
helm history my-app                              # View revision history

# Uninstall
helm uninstall my-app

# Debug / dry-run
helm template my-app ./chart                     # Render templates locally
helm install my-app ./chart --dry-run --debug    # Preview without installing
```

### Common Helm Charts (Pre-built Apps)
```
helm install nginx ingress-nginx/ingress-nginx    # Nginx Ingress Controller
helm install prometheus prometheus-community/kube-prometheus-stack  # Monitoring
helm install cert-manager jetstack/cert-manager    # TLS certificates
helm install argocd argo/argo-cd                   # GitOps CD
helm install redis bitnami/redis                   # Redis cache
```

---

## EKS DEEP DIVE

### EKS Architecture
```
┌─── EKS Cluster ─────────────────────────────────────────────┐
│                                                               │
│  ┌─── Control Plane (AWS Managed) ────────────────────────┐ │
│  │  API Server, etcd, Scheduler, Controller Manager       │ │
│  │  Multi-AZ, auto-scaled, patched by AWS                 │ │
│  │  You pay: $0.10/hour ($73/month)                       │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌─── Worker Nodes (YOU manage) ──────────────────────────┐ │
│  │                                                         │ │
│  │  Option 1: Managed Node Groups (recommended)           │ │
│  │  - EC2 instances managed by EKS                        │ │
│  │  - Auto-scaling, auto-patching                         │ │
│  │  - You choose instance type                            │ │
│  │                                                         │ │
│  │  Option 2: Fargate (serverless)                        │ │
│  │  - No EC2 to manage at all                             │ │
│  │  - Pay per pod (vCPU + memory)                         │ │
│  │  - Good for: batch jobs, infrequent workloads          │ │
│  │  - Limitations: no DaemonSets, no GPU                  │ │
│  │                                                         │ │
│  │  Option 3: Self-managed nodes                          │ │
│  │  - Full control, more work                             │ │
│  │                                                         │ │
│  └─────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
```

### EKS + AWS Integrations
```
┌──────────────────────┬───────────────────────────────────────┐
│ K8s Feature          │ AWS Integration                       │
├──────────────────────┼───────────────────────────────────────┤
│ LoadBalancer Service │ Creates AWS NLB automatically         │
│ Ingress              │ AWS ALB (via ALB Ingress Controller)  │
│ PersistentVolume     │ EBS (via EBS CSI Driver)              │
│ PV (ReadWriteMany)   │ EFS (via EFS CSI Driver)              │
│ Secrets              │ AWS Secrets Manager (External Secrets)│
│ IAM                  │ IRSA (IAM Roles for Service Accounts) │
│ Logging              │ CloudWatch (via Fluent Bit)           │
│ Monitoring           │ CloudWatch Container Insights         │
│ DNS                  │ Route 53 (via ExternalDNS)            │
│ TLS Certificates     │ ACM (via cert-manager)                │
└──────────────────────┴───────────────────────────────────────┘
```

### EKS Setup with eksctl
```bash
# Install eksctl
brew install eksctl                    # macOS

# Create cluster
eksctl create cluster \
  --name my-cluster \
  --region ca-central-1 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 5 \
  --managed

# Update kubeconfig
aws eks update-kubeconfig --name my-cluster --region ca-central-1

# Verify
kubectl get nodes

# Delete cluster (IMPORTANT — avoid charges!)
eksctl delete cluster --name my-cluster
```

---

## CHECKLIST
- [ ] Deploy app with Deployment + Service + Ingress
- [ ] Use ConfigMap and Secret in a pod
- [ ] Set up HPA for auto-scaling
- [ ] Install an app with Helm
- [ ] Create EKS cluster with eksctl, deploy app, then delete
