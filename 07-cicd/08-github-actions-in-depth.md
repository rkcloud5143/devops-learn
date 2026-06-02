# GitHub Actions — In-Depth Guide ⚡

Advanced patterns, OIDC, composite actions, self-hosted runners, and organization-wide workflows.

---

## Advanced Architecture

```
┌─── GitHub ──────────────────────────────────────────────────┐
│                                                              │
│  Event (push/PR/schedule/dispatch)                          │
│       │                                                      │
│       ▼                                                      │
│  Workflow Engine                                             │
│       │                                                      │
│       ├─→ GitHub-hosted runner (ubuntu-latest)              │
│       │     Fresh VM every run, 7GB RAM, 2 CPU              │
│       │     Free: 2000 min/month (public repos: unlimited)  │
│       │                                                      │
│       └─→ Self-hosted runner (your infrastructure)          │
│             EKS + Actions Runner Controller (ARC)           │
│             Persistent or ephemeral (recommended)           │
│             Custom tools pre-installed                       │
└──────────────────────────────────────────────────────────────┘
```

---

## OIDC Authentication (No Access Keys!)

```yaml
# OLD way (bad): Store AWS keys as GitHub secrets
# NEW way (good): OIDC — GitHub proves identity to AWS, gets temp creds

permissions:
  id-token: write    # Required for OIDC
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: ca-central-1
          # No access keys anywhere! AWS trusts GitHub's OIDC token
```

### How OIDC Works
```
1. GitHub Actions job starts
2. GitHub issues OIDC token (JWT) saying "I am repo X, branch Y, workflow Z"
3. Job sends token to AWS STS
4. AWS checks: "Does my IAM role trust this GitHub OIDC provider?"
5. If yes → AWS returns temporary credentials (15 min - 1 hour)
6. Job uses temp creds to access AWS

Benefits:
  • No long-lived credentials stored anywhere
  • Can restrict by repo, branch, environment
  • Credentials auto-expire
  • Audit trail in CloudTrail
```

### IAM Trust Policy for OIDC
```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
    },
    "StringLike": {
      "token.actions.githubusercontent.com:sub": "repo:myorg/my-repo:ref:refs/heads/main"
    }
  }
}
```

---

## Composite Actions (Reusable Steps)

```yaml
# actions/ecr-login/action.yaml
name: 'ECR Login'
description: 'Assume role and login to ECR'
inputs:
  role-arn:
    description: 'IAM role to assume'
    required: true
  region:
    description: 'AWS region'
    required: true
    default: 'ca-central-1'
  registry:
    description: 'ECR registry URL'
    required: true
outputs:
  registry:
    description: 'ECR registry URL'
    value: ${{ inputs.registry }}

runs:
  using: 'composite'
  steps:
    - uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ inputs.role-arn }}
        aws-region: ${{ inputs.region }}

    - name: Login to ECR
      shell: bash
      run: |
        aws ecr get-login-password --region ${{ inputs.region }} | \
          docker login --username AWS --password-stdin ${{ inputs.registry }}
```

### Using in workflows
```yaml
steps:
  - uses: myorg/shared-workflows/actions/ecr-login@main
    with:
      role-arn: arn:aws:iam::123456789012:role/ECRRole
      registry: 123456789012.dkr.ecr.ca-central-1.amazonaws.com
```

---

## Reusable Workflows (workflow_call)

```yaml
# .github/workflows/java-build.yaml (in shared-workflows repo)
name: Java Build
on:
  workflow_call:
    inputs:
      java-version:
        type: string
        default: '17'
      run-sonar:
        type: boolean
        default: true
      build-docker:
        type: boolean
        default: true
    secrets:
      SONAR_TOKEN:
        required: false

jobs:
  build:
    runs-on: arc-runners-java    # Self-hosted runner
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.java-version }}
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Build
        run: mvn clean package -DskipTests=${{ !inputs.run-sonar }}
      
      - name: SonarQube
        if: inputs.run-sonar
        run: mvn sonar:sonar -Dsonar.token=${{ secrets.SONAR_TOKEN }}
      
      - name: Docker Build & Push
        if: inputs.build-docker && github.ref == 'refs/heads/main'
        uses: myorg/shared-workflows/actions/docker-build@main
```

### Calling from any repo
```yaml
# In application repo: .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  build:
    uses: myorg/shared-workflows/.github/workflows/java-build.yaml@main
    with:
      run-sonar: true
      build-docker: true
    secrets: inherit    # Pass all org secrets automatically
```

---

## Self-Hosted Runners on EKS (ARC)

```
Actions Runner Controller (ARC):
  • Runs on your EKS cluster
  • Ephemeral runners (fresh pod per job, deleted after)
  • Scales to zero when no jobs (cost savings)
  • Custom images with your tools pre-installed
  • Multiple runner classes (java, node, docker)

Architecture:
  GitHub webhook → ARC controller → Spin up runner pod → Run job → Delete pod
```

### Installation
```bash
# Install ARC controller
helm install arc \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller \
  --namespace arc-systems --create-namespace

# Install runner scale set (one per runner class)
helm install arc-runners-java \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set \
  --namespace arc-runners --create-namespace \
  --set githubConfigUrl="https://github.com/myorg" \
  --set githubConfigSecret.github_token="$GITHUB_TOKEN" \
  --set maxRunners=10 \
  --set minRunners=0 \
  --set template.spec.containers[0].image="myorg/runner-java:latest"
```

### Custom Runner Image
```dockerfile
FROM ghcr.io/actions/actions-runner:latest

# Install tools needed for builds
RUN sudo apt-get update && sudo apt-get install -y \
    docker.io maven openjdk-17-jdk \
    awscli kubectl helm terraform \
    trivy trufflehog hadolint \
    && sudo rm -rf /var/lib/apt/lists/*
```

---

## Advanced Patterns

### Monorepo — Only Build What Changed
```yaml
on:
  push:
    paths:
      - 'services/api/**'
      - '!services/api/README.md'

# Or dynamic with dorny/paths-filter:
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      api: ${{ steps.filter.outputs.api }}
      frontend: ${{ steps.filter.outputs.frontend }}
    steps:
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            api:
              - 'services/api/**'
            frontend:
              - 'services/frontend/**'

  build-api:
    needs: changes
    if: needs.changes.outputs.api == 'true'
    uses: ./.github/workflows/build.yml
```

### Environment Protection + Deployment
```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment:
      name: production              # Requires approval in GitHub settings
      url: https://myapp.com
    steps:
      - run: helm upgrade --install myapp ./chart
```

### Concurrency (Prevent Duplicate Runs)
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true    # Cancel older runs when new push arrives
```

### Artifacts Between Jobs
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: docker build -t myapp .
      - run: docker save myapp > image.tar
      - uses: actions/upload-artifact@v4
        with:
          name: docker-image
          path: image.tar

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: docker-image
      - run: docker load < image.tar
```

### GitOps Update (CI triggers ArgoCD)
```yaml
- name: Update GitOps repo
  uses: myorg/shared-workflows/actions/gitops-update@main
  with:
    image: myapp
    tag: ${{ github.sha }}
    environment: qa
    gh-token: ${{ secrets.GITOPS_TOKEN }}
# This updates images.yaml in gitops repo → ArgoCD detects → deploys
```

---

## GitHub Actions vs Jenkins — Deep Comparison

| Feature | GitHub Actions | Jenkins |
|---------|---------------|---------|
| Config | YAML in repo | Groovy Jenkinsfile |
| Infrastructure | Managed or self-hosted | Always self-hosted |
| Scaling | Auto (ARC) | Manual (add agents) |
| Secrets | GitHub Secrets + OIDC | Credentials plugin |
| Plugins/Extensions | Marketplace (actions) | Plugin ecosystem (1800+) |
| Multi-repo orchestration | Limited (workflow_dispatch) | Strong (upstream/downstream) |
| Approval gates | Environment protection | input step |
| Visibility | PR checks, status badges | Dashboard, Blue Ocean |
| Cost | Free tier + $0.008/min | Free (but server costs) |
| Debugging | Re-run with debug logging | Console output, SSH to agent |
| Cron/scheduled | schedule trigger (cron) | Build periodically |
| Parameterized builds | workflow_dispatch inputs | parameters {} |

---

*GitHub Actions = the modern standard for CI/CD. Master shared workflows and OIDC! ⚡*
