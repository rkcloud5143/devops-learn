# Real-World DevOps Experience 🛠️

Documented projects and work from a Senior DevOps Engineer role. Use as reference for interviews, resume, or learning.

> **Note:** Company names replaced with placeholders for public sharing.

---

## 🏢 Environment Overview

```
Company: FinServ Corp (Financial Services, Canada)
Team: DevOps / DevSecOps (3-5 engineers)
Scale: 100+ EC2 instances, 4 EKS clusters, 2 AWS accounts, multi-region

AWS Accounts:
  • Production (us-west-1, us-east-1)
  • NonProd (us-east-1)
  • Infrastructure/Tools (us-west-1)
  • EU (eu-west-2)

IaC Stack:
  • Terraform + Terragrunt (infra-terraform repo)
  • Ansible (infra-ansible repo)
  • Helm + ArgoCD (infra-gitops repo)
  • GitHub Actions shared workflows (shared-workflows repo)
  • Jenkins (legacy CI, migrating to GHA)
  • Cloudflare Zero Trust (infra-cloudflare repo)
```

---

## 🌐 AWS Transit Gateway Migration (from VPC Peering)

### Problem
- 48+ VPC peering connections across 19 VPCs in 4 AWS accounts
- Spaghetti routing — hard to audit, hard to troubleshoot
- Adding a new VPC required creating peering to every other VPC
- Blackhole routes accumulating from deleted/failed peerings

### What I Did
- Mapped ALL existing peering connections and routes across 4 accounts
- Identified and documented blackhole/orphaned routes
- Designed Transit Gateway hub-and-spoke architecture
- Created deletion plan with rollback procedures
- Migrated inter-VPC traffic to TGW (multi-region)
- Verified Route53 private hosted zone associations post-migration
- Cleaned up 48 peering connections and associated route table entries

### Architecture (Before → After)
```
BEFORE (Mesh — 48 connections):
  VPC-A ↔ VPC-B ↔ VPC-C
    ↕         ↕         ↕
  VPC-D ↔ VPC-E ↔ VPC-F
  (N*(N-1)/2 = exponential growth)

AFTER (Hub & Spoke):
  VPC-A ─┐
  VPC-B ─┼── Transit Gateway ──┼── VPC-D
  VPC-C ─┘                     └── VPC-E
  (N connections = linear growth)
```

### Key Deliverables
- 42 documentation files (route mappings, deletion plans, verification reports)
- Zero-downtime migration
- Route53 private hosted zone re-association
- Cross-region TGW peering (us-east-1 ↔ us-west-1 ↔ eu-west-2)

---

## 🔄 GitOps with ArgoCD

### Architecture
```
Developer pushes code
      │
      ▼
GitHub Actions (CI):
  Build → Test → Scan → Push image to ECR
      │
      ▼
gitops-update action:
  Updates images.yaml in infra-gitops repo
      │
      ▼
ArgoCD (CD):
  Detects change → Syncs to Kubernetes cluster
      │
      ▼
App deployed (zero-touch from developer perspective)
```

### What I Built
```
infra-gitops/
├── argocd-assets/
│   ├── app-of-apps.yaml          # Root application (discovers all apps)
│   ├── applications/              # App manifests (ApplicationSet for multi-cluster)
│   ├── clusters/                  # EKS cluster connection secrets (SOPS encrypted)
│   └── repositories/              # Git repo credentials
├── helm-charts/
│   ├── java-apps/                 # Java microservices chart
│   ├── jenkins/                   # Jenkins controller + dynamic agents
│   ├── datadog/                   # Monitoring (DaemonSet on all nodes)
│   ├── karpenter/                 # Node autoscaling
│   ├── external-secrets/          # AWS Secrets Manager → K8s Secrets
│   ├── awx/                       # AWX (Ansible automation)
│   ├── gha-runner-scale-set/      # GitHub Actions self-hosted runners
│   └── risk-models/               # ML model deployments
├── secrets-manager/               # SOPS encrypted secrets
└── images.yaml                    # Single source of truth for all image tags
```

### 4 EKS Clusters Managed
| Cluster | Region | Purpose |
|---------|--------|---------|
| devops | us-west-1 | Jenkins, GHA runners, infra tools |
| nonprod | us-east-1 | QA/staging applications |
| ml-qa | us-east-1 | ML model testing |
| ml-prod | us-west-1 | Production ML models |

### Key Patterns Used
- **App-of-Apps** — One root Application discovers all apps automatically
- **ApplicationSet** — Deploy same chart to multiple clusters with different values
- **SOPS + KMS** — Encrypt secrets in Git, decrypt at deploy time
- **Karpenter** — Auto-provision right-sized nodes (Spot + On-Demand)
- **External Secrets Operator** — Pull secrets from AWS Secrets Manager into K8s
- **Node scheduling strategy** — System apps on managed nodes, workloads on Karpenter nodes

---

## ⚙️ GitHub Actions Shared Workflows

### Problem
- 50+ repos, each with its own CI/CD pipeline
- Inconsistent builds, duplicated logic, hard to maintain
- Security scanning not standardized

### What I Built
```
shared-workflows/
├── .github/workflows/              # Shared workflows (workflow_call)
│   ├── java-build.yaml             # Maven + SonarQube + Docker + GitOps
│   ├── cog-build.yaml              # Cog ML container builds
│   └── node-build.yaml             # Node.js builds
├── actions/                         # Reusable composite actions
│   ├── aws-credentials/             # OIDC auth (no access keys!)
│   ├── docker-build/                # Build Docker image
│   ├── ecr-login-qa/                # Assume role + login QA ECR
│   ├── ecr-login-prod/              # Assume role + login Prod ECR
│   ├── ecr-login-infra/             # Login Infra ECR
│   ├── ecr-push/                    # Tag + push image
│   ├── gitops-update/               # Update image in infra-gitops → ArgoCD deploys
│   ├── container-tag/               # Generate tag: {branch}-{sha}-{date}-{build}
│   ├── sonar-credentials/           # Fetch SonarQube creds from SSM
│   ├── sonar-analysis/              # Run SonarQube scan
│   ├── maven-settings/              # Org Maven settings.xml
│   └── maven-toolchains/            # JDK 17 toolchains
├── runner-images/                    # Custom ARC runner Dockerfiles
│   ├── java/                         # JDK + Maven + Docker
│   └── cog/                          # Cog ML runtime
└── Makefile                          # Build runner images, lint
```

### How Teams Use It
```yaml
# In any repo's .github/workflows/build.yml — just 10 lines:
name: Build
on: [push, pull_request]
jobs:
  build:
    uses: org/shared-workflows/.github/workflows/java-build.yaml@main
    with:
      run_sonar: true
      build_docker: true
      single_qa_env: true
    secrets: inherit
```

### Self-Hosted Runners on EKS
- **GitHub Actions Runner Controller (ARC)** on EKS
- Runners scale to zero when idle (cost savings)
- Custom runner images (Java, Cog) pre-loaded with tools
- Karpenter provisions nodes on demand

---

## 🤖 AWX (Ansible Tower on Kubernetes)

### What is AWX?
Web UI + API + scheduler for running Ansible playbooks. Like Jenkins but for configuration management.

### What I Did
- Deployed AWX on EKS using AWX Operator (Helm)
- Custom AWX instance with EFS persistent storage
- Integrated with GitHub repos (auto-sync projects on branch push)
- Webhook proxy setup for GitHub → AWX job triggers
- Used for: server patching, config management, one-off automation

### Architecture
```
GitHub push → Webhook → AWX → Runs Ansible playbook → Configures servers
```

---

## 🔒 Cloudflare Zero Trust (IaC)

### Problem
- 170+ gateway network policies managed via dashboard (click-ops)
- 40+ gateway lists (IPs, emails, hostnames)
- No audit trail, inconsistent naming, impossible to review changes
- Policies referenced AD groups by UUID (unreadable)

### What I Built
```
infra-cloudflare/
├── data/zero-trust/
│   ├── lists/
│   │   ├── emails/         # Team membership lists (JSON)
│   │   ├── ips/            # IP allow/block lists
│   │   ├── hostnames/      # Domain lists
│   │   └── serials/        # Device serial numbers
│   └── policies/
│       ├── database-access/ # DB access rules
│       ├── ssh-access/      # SSH/SFTP rules
│       ├── app-access/      # Application access
│       ├── block-rules/     # Block policies
│       └── egress/          # Egress policies
├── infra/
│   ├── modules/
│   │   ├── tf-gateway-list/    # Terraform module for lists
│   │   ├── tf-gateway-policy/  # Terraform module for policies
│   │   ├── tf-device-profile/  # Read-only (phase 2)
│   │   └── tf-tunnel-route/    # Read-only (phase 2)
│   └── Accounts/PropelHoldings/zero-trust/
│       ├── lists/terragrunt.hcl
│       └── policies/terragrunt.hcl
├── Jenkinsfile             # CI: terraform plan on PR
└── Makefile
```

### Key Design Decisions
- **Separation of data and code** — IT/DevOps edit JSON files, platform team owns Terraform modules
- **API token security** — IP-restricted to VPN + Jenkins NAT, stored in AWS Secrets Manager
- **Imported 170 existing rules** into Terraform state
- **Naming convention** — `Action-Category-Team-Environment-Description`
- **Phase 2** — Device profiles and tunnels in read-only mode (ignore_changes = all)

---

## 🩹 AWS Systems Manager Patch Management

### Problem
- Vendor previously managed patching — they left with no documentation
- 100+ EC2 instances across 2 accounts, multiple server roles
- No visibility into patch compliance
- Need backup before patching (in case of rollback)

### What I Built
- **Terraform modules** for SSM Patch Manager (baselines, maintenance windows, targets)
- **Patch groups by role**: CI, dev, QA, sandbox, DB, devops/infra
- **Scheduled maintenance windows** per environment
- **Pre-patch AMI backup** (automatic snapshot before patching)
- **Lambda function** for patch compliance reporting → Slack notifications
- **Jira integration** for prod patch change management tickets

### Architecture
```
Terraform defines:
  Patch Baselines → Maintenance Windows → Targets (by tag)
                                              │
                                              ▼
                                    SSM runs patches
                                              │
                                              ▼
                              Lambda checks compliance
                                              │
                                              ▼
                              Slack notification + Jira ticket
```

---

## 🛡️ Jenkins Security Scanning Pipeline

### Problem
- Jenkins only did basic syntax validation
- 1000+ security issues across repos (no enforcement)
- No standardized scanning for IaC, containers, or secrets

### What I Built
Introduced multi-level scanning in Jenkinsfile:

```
Level 1 (Basic — always run):
  • yamllint — YAML syntax
  • terraform validate — TF syntax
  • helm template + kubeconform — K8s manifest validation

Level 2 (Medium — fail build):
  • trufflehog — Secret scanning (git history)
  • hadolint — Dockerfile best practices
  • shellcheck — Bash script analysis
  • actionlint — GitHub Actions workflow validation

Level 3 (Advanced):
  • tfsec / checkov — Terraform security scanning
  • trivy — Container image vulnerabilities
  • SonarQube — Code quality + coverage
```

---

## 🖥️ EC2 Migration: Amazon Linux 2 → Amazon Linux 2023

### What I Did
- Migrated production servers from AL2 (EOL) to AL2023
- Created Terraform modules for new EC2 instances (previously manually created)
- Used CIS-hardened AMIs
- Preserved EIPs (vendor whitelisted IPs)
- Ansible for post-provisioning configuration
- Datadog agent deployment via Ansible role

### Process
```
1. Terraform creates new AL2023 instance (CIS hardened AMI)
2. Ansible configures: Docker, monitoring, app dependencies
3. Test application on new instance
4. Swap EIP from old → new instance
5. Monitor for issues
6. Decommission old AL2 instance
```

---

## 📊 Datadog Monitoring & RBAC

### What I Did
- Designed role-based access for Datadog (L1 Developer, Standard, Admin)
- 54 users with Standard Role — reviewed and right-sized permissions
- Set up MCP (Model Context Protocol) integration for CLI access
- Monitor management and alert routing

---

## 🏦 AWS IAM Identity Center (SSO)

### What I Did
- Managed SSO policies via Terraform (tf-sso-policies module)
- Created tiered developer access: L1 (read-only) → L2 (limited write)
- Permission boundaries preventing privilege escalation
- SSM Session Manager for EC2 access (no SSH keys)
- Session logging to S3 buckets (per-region)

---

## 🧹 GitHub Repository Hygiene

### Problem
- One repo had 883 stale branches (impacting CI/CD performance)
- Build failures from branch count

### What I Did
- Automated stale branch cleanup script (with 0.5s rate limiting)
- Enabled auto-delete of merged branches across all repos
- Posted org-wide communication for teams to clean their branches
- Reduced branch counts from 883 → manageable levels

---

## 📋 Skills Matrix (From This Experience)

```
Category              Technologies
────────────          ────────────────────────────────────────────
Cloud                 AWS (EC2, EKS, VPC, RDS, S3, IAM, SSM, Lambda,
                      Route53, TGW, Secrets Manager, Organizations)
IaC                   Terraform, Terragrunt, Packer, CloudFormation
Config Mgmt           Ansible, AWX (Ansible Tower)
Containers            Docker, ECR, EKS, Helm, Karpenter
Orchestration         Kubernetes, ArgoCD, Kustomize
CI/CD                 GitHub Actions (shared workflows, ARC runners),
                      Jenkins (Declarative pipelines, shared libraries)
Security              Cloudflare Zero Trust, WAF, TruffleHog, tfsec,
                      Trivy, SonarQube, CIS hardened AMIs
Monitoring            Datadog (APM, Logs, Infra), CloudWatch, Prometheus
Networking            Transit Gateway, VPC Peering, Direct Connect,
                      Cloudflare Tunnels, Route53, ALB/NLB
Scripting             Bash, Python (boto3), Makefiles
OS                    Amazon Linux 2/2023, Ubuntu
Version Control       Git, GitHub (CODEOWNERS, branch protection,
                      Copilot PR reviews, organization management)
```

---

*This is real-world experience — not textbook knowledge. Use it to answer "Tell me about a time when..." questions! 🛠️*
