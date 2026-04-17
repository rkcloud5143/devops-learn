# ArgoCD — GitOps for Kubernetes 🔄

Deploy to Kubernetes by pushing to Git. No more `kubectl apply`.

---

## What is ArgoCD?

```
Traditional CI/CD (Push-based):
  CI builds image → CD runs kubectl apply → Cluster updated
  Problem: CI needs cluster credentials 😬

GitOps with ArgoCD (Pull-based):
  CI builds image → Updates Git manifest → ArgoCD detects change → Cluster syncs
  Benefit: CI never touches the cluster 🔒

┌─── Git Repo ──────────┐     ┌─── ArgoCD (in cluster) ──┐     ┌─── K8s Cluster ──┐
│                        │     │                           │     │                   │
│  deployment.yaml       │────▶│  Watches Git repo         │────▶│  Syncs resources  │
│  service.yaml          │     │  Compares with cluster    │     │  to match Git     │
│  values.yaml           │     │  Auto-syncs or manual     │     │                   │
│                        │     │  Detects drift            │     │                   │
└────────────────────────┘     └───────────────────────────┘     └───────────────────┘
```

**Pizza shop:** Instead of the manager calling each kitchen with instructions, they update the recipe book (Git). Each kitchen (cluster) has a copy and automatically follows the latest version.

---

## How ArgoCD Works

```
1. You define an "Application" in ArgoCD
   - Source: Git repo + path + branch
   - Destination: Kubernetes cluster + namespace

2. ArgoCD continuously watches the Git repo

3. When Git changes:
   - ArgoCD compares Git state vs cluster state
   - Shows diff (what changed)
   - Syncs automatically (or waits for approval)

4. If someone changes the cluster directly (drift):
   - ArgoCD detects it
   - Shows "OutOfSync"
   - Can auto-correct back to Git state
```

---

## Install ArgoCD on EKS

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Or with Helm (recommended)
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace \
    --set server.service.type=LoadBalancer

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Install ArgoCD CLI
brew install argocd

# Login
argocd login <ARGOCD_SERVER> --username admin --password <PASSWORD>
```

---

## ArgoCD Application

### Via CLI
```bash
argocd app create my-app \
    --repo https://github.com/myorg/k8s-manifests.git \
    --path apps/my-app/production \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace production \
    --sync-policy automated \
    --auto-prune \
    --self-heal
```

### Via YAML (Recommended — App of Apps pattern)
```yaml
# argocd/applications/my-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/myorg/k8s-manifests.git
    targetRevision: main
    path: apps/my-app/production

    # If using Helm chart:
    # helm:
    #   valueFiles:
    #     - values-prod.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true       # Delete resources removed from Git
      selfHeal: true    # Fix drift automatically
    syncOptions:
      - CreateNamespace=true
```

---

## GitOps Repo Structure

### Separate Repos (Recommended)

```
app-code/                    ← Application source code
├── src/
├── Dockerfile
├── .github/workflows/
│   └── ci.yml              ← CI: build + push image
└── README.md

k8s-manifests/               ← Kubernetes manifests (ArgoCD watches this)
├── apps/
│   ├── my-app/
│   │   ├── base/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   ├── dev/
│   │   │   ├── kustomization.yaml
│   │   │   └── patch.yaml
│   │   └── production/
│   │       ├── kustomization.yaml
│   │       └── patch.yaml
│   └── another-app/
│       └── ...
├── argocd/
│   └── applications/        ← ArgoCD Application definitions
│       ├── my-app.yaml
│       └── another-app.yaml
└── README.md
```

### The Full GitOps Flow

```
Developer pushes code to app-code repo
        │
        ▼
GitHub Actions (CI):
  1. Run tests
  2. Build Docker image
  3. Push to ECR (tag: git-sha)
  4. Update k8s-manifests repo with new image tag  ← KEY STEP
        │
        ▼
k8s-manifests repo updated
        │
        ▼
ArgoCD detects change
        │
        ▼
ArgoCD syncs → Kubernetes updated
        │
        ▼
Rollback? → git revert in k8s-manifests repo → ArgoCD syncs old version
```

### CI Pipeline That Updates Manifests
```yaml
# In app-code/.github/workflows/ci.yml
- name: Update manifest repo
  run: |
    git clone https://x-access-token:${{ secrets.MANIFEST_REPO_TOKEN }}@github.com/myorg/k8s-manifests.git
    cd k8s-manifests
    # Update image tag in deployment
    sed -i "s|image:.*|image: $ECR_REGISTRY/my-app:${{ github.sha }}|" apps/my-app/base/deployment.yaml
    git config user.name "github-actions"
    git config user.email "actions@github.com"
    git add .
    git commit -m "Update my-app image to ${{ github.sha }}"
    git push
```

---

## ArgoCD Commands

```bash
# List applications
argocd app list

# Get app details
argocd app get my-app

# Sync (deploy)
argocd app sync my-app

# Rollback
argocd app rollback my-app

# View history
argocd app history my-app

# Diff (what would change)
argocd app diff my-app

# Delete app (doesn't delete K8s resources by default)
argocd app delete my-app
```

---

## ArgoCD vs Traditional CI/CD

| Feature | Traditional (Push) | ArgoCD (Pull/GitOps) |
|---------|-------------------|---------------------|
| Source of truth | CI pipeline | Git repository |
| Cluster access | CI needs kubeconfig | Only ArgoCD (in-cluster) |
| Drift detection | None | Automatic |
| Rollback | Re-run pipeline | `git revert` |
| Audit trail | CI logs | Git history |
| Security | CI has cluster creds | CI never touches cluster |
| Multi-cluster | Complex | Built-in |

---

*ArgoCD = Git is the truth, cluster follows. Simple, secure, auditable! 🔄*
