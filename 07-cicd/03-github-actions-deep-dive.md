# GitHub Actions — Complete Guide 🔧

Everything you need to know about GitHub Actions for CI/CD.

---

## How GitHub Actions Works

```
Code Push / PR
      │
      ▼
GitHub detects event
      │
      ▼
Reads .github/workflows/*.yml
      │
      ▼
Spins up runner (Ubuntu VM)
      │
      ▼
Executes jobs & steps
      │
      ▼
Reports status (✅ or ❌)
```

---

## Core Concepts

```
Workflow (.yml file)
│
├── Event (trigger)          ← What starts the pipeline
│   ├── push
│   ├── pull_request
│   ├── schedule (cron)
│   ├── workflow_dispatch    ← Manual trigger
│   └── release
│
├── Jobs                     ← Groups of steps (parallel by default)
│   ├── Job 1: build
│   ├── Job 2: test
│   └── Job 3: deploy (needs: [build, test])
│
├── Steps                    ← Individual tasks within a job
│   ├── uses: actions/checkout@v4    ← Pre-built action
│   └── run: npm test               ← Shell command
│
├── Secrets                  ← Encrypted variables
│   ├── Settings → Secrets → Actions
│   └── Referenced as ${{ secrets.NAME }}
│
└── Runners                  ← Machines that execute jobs
    ├── GitHub-hosted (ubuntu-latest, windows-latest, macos-latest)
    └── Self-hosted (your own machines)
```

---

## Real-World Pipeline: Build → Test → Deploy to EKS

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  AWS_REGION: ca-central-1
  ECR_REPOSITORY: my-app
  EKS_CLUSTER: my-cluster

# Use OIDC for AWS auth (no access keys!)
permissions:
  id-token: write
  contents: read

jobs:
  # ─── JOB 1: Build & Test ───
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Run security scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'

  # ─── JOB 2: Build & Push Docker Image ───
  docker:
    needs: build
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.build-image.outputs.image }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Scan Docker image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.build-image.outputs.image }}
          severity: 'CRITICAL,HIGH'

  # ─── JOB 3: Deploy to EKS ───
  deploy:
    needs: docker
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Only deploy from main
    environment: production               # Requires approval
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: ${{ env.AWS_REGION }}

      - name: Update kubeconfig
        run: aws eks update-kubeconfig --name $EKS_CLUSTER --region $AWS_REGION

      - name: Deploy with Helm
        run: |
          helm upgrade --install my-app ./helm/my-app \
            --namespace production \
            --set image.repository=${{ needs.docker.outputs.image }} \
            --wait --timeout 5m

      - name: Verify deployment
        run: |
          kubectl rollout status deployment/my-app -n production --timeout=5m
```

---

## Useful Patterns

### Matrix Strategy (Test Multiple Versions)
```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

### Caching Dependencies
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: ${{ runner.os }}-npm-
```

### Manual Trigger with Inputs
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy to which environment?'
        required: true
        type: choice
        options: [dev, staging, prod]
```

### Reusable Workflows
```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      AWS_ROLE_ARN:
        required: true

# Called from another workflow:
jobs:
  deploy-dev:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: dev
    secrets:
      AWS_ROLE_ARN: ${{ secrets.DEV_AWS_ROLE_ARN }}
```

### Path Filters (Monorepo)
```yaml
on:
  push:
    paths:
      - 'services/api/**'      # Only trigger when API code changes
      - '!services/api/README.md'  # Ignore README changes
```

### Environment Protection Rules
```yaml
jobs:
  deploy-prod:
    environment:
      name: production
      url: https://myapp.com
    # In GitHub Settings → Environments → production:
    # - Required reviewers: 2
    # - Wait timer: 5 minutes
    # - Deployment branches: main only
```

---

## GitHub Actions vs Jenkins — Quick Comparison

| Feature | GitHub Actions | Jenkins |
|---------|---------------|---------|
| Hosting | GitHub-hosted or self-hosted | Self-hosted only |
| Config | YAML in repo | Jenkinsfile or UI |
| Setup | Zero — built into GitHub | Install, configure, maintain |
| Cost | Free tier (2000 min/month) | Free (but you pay for servers) |
| Plugins | Marketplace actions | Huge plugin ecosystem |
| Scaling | Auto (GitHub-hosted) | Manual (add agents) |
| Best for | GitHub repos, modern teams | Enterprise, complex pipelines |

---

*GitHub Actions is the easiest way to start with CI/CD! 🚀*
