# Standard File Names & Paths — DevOps Convention Guide 📁

Where things go and what they're called. These are industry standards — don't rename them.

---

## Docker

```
my-app/
├── Dockerfile                         # Default (docker build looks for this)
├── Dockerfile.dev                     # Dev-specific
├── Dockerfile.prod                    # Prod-specific
├── .dockerignore                      # Files excluded from build context
├── docker-compose.yml                 # Default Compose file
├── docker-compose.override.yml        # Auto-merged with docker-compose.yml
├── docker-compose.dev.yml             # Dev overrides
├── docker-compose.prod.yml            # Prod overrides
└── .env                               # Default env vars for Compose
```

```bash
# Docker uses these by default — no flags needed
docker build .                         # Reads ./Dockerfile
docker compose up                      # Reads ./docker-compose.yml

# Custom Dockerfile
docker build -f Dockerfile.prod .

# Custom Compose file
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## GitHub Actions

```
my-app/
└── .github/
    └── workflows/                     # MUST be this exact path
        ├── ci.yml                     # Any .yml or .yaml file here = a workflow
        ├── cd.yml
        ├── deploy.yml
        ├── test.yml
        ├── release.yml
        └── cron-cleanup.yml

    # Other GitHub config files
    ├── CODEOWNERS                     # PR review assignments
    ├── dependabot.yml                 # Dependency updates
    ├── PULL_REQUEST_TEMPLATE.md       # PR template
    └── ISSUE_TEMPLATE/
        └── bug_report.md
```

**Rules:**
- Path MUST be `.github/workflows/` (case-sensitive)
- Files MUST be `.yml` or `.yaml`
- File names can be anything — GitHub runs ALL workflows in that folder
- Each file = one workflow (can have multiple jobs)

```yaml
# .github/workflows/ci.yml — minimal example
name: CI                               # Display name (optional)
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

### Common Workflow File Names

| File | Purpose |
|------|---------|
| `ci.yml` | Build + test on every push/PR |
| `cd.yml` | Deploy to environments |
| `deploy.yml` | Combined CI/CD |
| `release.yml` | Create releases/tags |
| `test.yml` | Tests only |
| `lint.yml` | Linting/formatting checks |
| `security.yml` | Security scanning (SAST, dependency audit) |
| `docker-publish.yml` | Build and push Docker image |
| `terraform.yml` | Terraform plan/apply |
| `cron-cleanup.yml` | Scheduled maintenance tasks |

---

## Jenkins

```
my-app/
├── Jenkinsfile                        # Default (Pipeline looks for this)
├── Jenkinsfile.dev                    # Environment-specific (configure in Jenkins)
├── Jenkinsfile.prod
└── jenkins/
    ├── scripts/
    │   ├── build.sh
    │   └── deploy.sh
    └── agents/
        └── Dockerfile                 # Custom Jenkins agent image
```

**Rules:**
- Default file: `Jenkinsfile` (capital J, no extension)
- Must be in repo root (or configure path in Jenkins)
- Written in Groovy (Declarative or Scripted syntax)

```groovy
// Jenkinsfile — minimal example
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
}
```

### Jenkins on Server

```
/var/lib/jenkins/                      # JENKINS_HOME
├── config.xml                         # Global config
├── jobs/                              # Job definitions
│   └── my-app/
│       └── config.xml
├── workspace/                         # Build workspaces
├── plugins/                           # Installed plugins
├── secrets/                           # Encrypted secrets
├── users/                             # User configs
└── nodes/                             # Agent configs
```

---

## Terraform

```
my-module/
├── main.tf                            # Primary resources
├── variables.tf                       # Input variables
├── outputs.tf                         # Output values
├── providers.tf                       # Provider configuration
├── backend.tf                         # State backend config
├── versions.tf                        # Required provider versions
├── data.tf                            # Data sources
├── locals.tf                          # Local values
├── terraform.tfvars                   # Variable values (default loaded)
├── prod.tfvars                        # Environment-specific values
├── dev.tfvars
├── .terraform/                        # Downloaded providers/modules (gitignore)
├── .terraform.lock.hcl                # Provider lock file (commit to git)
├── terraform.tfstate                  # State file (gitignore — use remote)
├── terraform.tfstate.backup           # State backup (gitignore)
└── .gitignore
```

**Naming conventions:**
- `main.tf` — primary resources (or split by resource type: `ec2.tf`, `vpc.tf`, `rds.tf`)
- `variables.tf` — ALL input variables
- `outputs.tf` — ALL outputs
- `*.auto.tfvars` — auto-loaded variable files (no `-var-file` flag needed)

---

## Terragrunt

```
environments/
├── terragrunt.hcl                     # Root config (backend + provider)
├── dev/
│   ├── env.hcl                        # Environment-level variables
│   └── vpc/
│       └── terragrunt.hcl             # Module config
└── prod/
    ├── env.hcl
    └── vpc/
        └── terragrunt.hcl
```

**Rules:**
- Default file: `terragrunt.hcl`
- `find_in_parent_folders()` walks up looking for `terragrunt.hcl`

---

## Kubernetes

```
k8s/
├── deployment.yaml                    # or .yml — both work
├── service.yaml
├── ingress.yaml
├── configmap.yaml
├── secret.yaml
├── hpa.yaml
├── pdb.yaml
├── namespace.yaml
├── serviceaccount.yaml
└── kustomization.yaml                 # Kustomize config (exact name required)
```

### Helm Chart

```
my-app/
├── Chart.yaml                         # REQUIRED — chart metadata (exact name)
├── Chart.lock                         # Dependency lock file
├── values.yaml                        # REQUIRED — default values (exact name)
├── values-dev.yaml                    # Environment overrides
├── values-staging.yaml
├── values-prod.yaml
├── templates/                         # REQUIRED — template directory (exact name)
│   ├── _helpers.tpl                   # Template helpers (underscore = not rendered)
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   ├── serviceaccount.yaml
│   ├── NOTES.txt                      # Post-install message
│   └── tests/
│       └── test-connection.yaml
└── charts/                            # Sub-chart dependencies
```

**Helm rules:**
- `Chart.yaml` — exact name, capital C
- `values.yaml` — exact name, lowercase
- `templates/` — exact directory name
- Files starting with `_` in templates/ are helpers, not rendered

---

## ArgoCD

```
k8s-manifests/                         # GitOps repo
├── apps/
│   └── my-app/
│       ├── base/
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   └── kustomization.yaml
│       ├── overlays/
│       │   ├── dev/
│       │   │   └── kustomization.yaml
│       │   └── prod/
│       │       └── kustomization.yaml
│       └── Chart.yaml                 # If using Helm
└── argocd/
    └── applications/                  # ArgoCD Application CRDs
        ├── my-app-dev.yaml
        └── my-app-prod.yaml
```

---

## Ansible

```
ansible/
├── ansible.cfg                        # Ansible config (exact name)
├── inventory/
│   ├── hosts.ini                      # or hosts.yml
│   ├── dev/
│   │   └── hosts.ini
│   └── prod/
│       └── hosts.ini
├── playbooks/
│   ├── site.yml                       # Main playbook
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   └── nginx/
│       ├── tasks/main.yml             # REQUIRED (exact path)
│       ├── handlers/main.yml
│       ├── templates/
│       ├── files/
│       ├── vars/main.yml
│       ├── defaults/main.yml
│       └── meta/main.yml
├── group_vars/
│   ├── all.yml                        # Variables for all hosts
│   └── webservers.yml                 # Variables for webservers group
├── host_vars/
│   └── web1.example.com.yml
└── requirements.yml                   # Role dependencies
```

---

## Git

```
my-repo/
├── .git/                              # Git internals (don't touch)
├── .gitignore                         # Files to ignore
├── .gitattributes                     # File handling rules
├── .gitmodules                        # Submodule config
├── README.md                          # Project documentation
├── LICENSE                            # License file
├── CHANGELOG.md                       # Version history
└── CONTRIBUTING.md                    # Contribution guidelines
```

---

## SOPS

```
my-repo/
├── .sops.yaml                         # SOPS config (exact name, in repo root)
├── secrets.enc.yaml                   # Encrypted secrets
└── secrets.dec.yaml                   # Decrypted (GITIGNORE THIS)
```

---

## Standard .gitignore for DevOps

```gitignore
# Terraform
.terraform/
*.tfstate
*.tfstate.backup
*.tfplan
crash.log
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Terragrunt
.terragrunt-cache/

# Docker
docker-compose.override.yml

# Secrets
.env
*.pem
*.key
*secret*
*decrypted*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Python
__pycache__/
*.pyc
.venv/

# Node
node_modules/
```

---

## Quick Reference Table

| Tool | Default File | Must Be Exact? |
|------|-------------|----------------|
| Docker | `Dockerfile` | Yes (or use `-f`) |
| Docker Compose | `docker-compose.yml` | Yes (or use `-f`) |
| Docker ignore | `.dockerignore` | Yes |
| GitHub Actions | `.github/workflows/*.yml` | Path must be exact |
| Jenkins | `Jenkinsfile` | Yes (capital J, no extension) |
| Terraform | `*.tf` in current dir | Any `.tf` name works |
| Terraform vars | `terraform.tfvars` / `*.auto.tfvars` | Auto-loaded names |
| Terraform lock | `.terraform.lock.hcl` | Yes |
| Terragrunt | `terragrunt.hcl` | Yes |
| Helm chart | `Chart.yaml` | Yes (capital C) |
| Helm values | `values.yaml` | Yes |
| Helm templates | `templates/` dir | Yes |
| Kustomize | `kustomization.yaml` | Yes |
| Ansible config | `ansible.cfg` | Yes |
| Ansible role tasks | `roles/*/tasks/main.yml` | Yes |
| SOPS config | `.sops.yaml` | Yes |
| Git ignore | `.gitignore` | Yes |

---

*Bookmark this — you'll reference it every time you set up a new project! 📁*
