# CI/CD — GitHub Actions

## Repository Structure
```
my-app/
├── .github/
│   └── workflows/
│       └── deploy.yml    ← Pipeline definition
├── Dockerfile
├── src/
└── terraform/
```

## Example Workflow
```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  AWS_REGION: ca-central-1
  ECR_REPOSITORY: my-app

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster my-cluster \
            --service my-service \
            --force-new-deployment
```

## Key Concepts
```
Workflow (.yml file)
├── Trigger (on: push, pull_request, schedule)
├── Jobs (run in parallel by default)
│   ├── Job 1: build
│   │   ├── Step 1: checkout
│   │   ├── Step 2: test
│   │   └── Step 3: build image
│   └── Job 2: deploy (needs: build)
│       ├── Step 1: configure AWS
│       └── Step 2: deploy
└── Secrets (stored in GitHub Settings → Secrets)
    ├── AWS_ACCESS_KEY_ID
    └── AWS_SECRET_ACCESS_KEY
```

## Checklist
- [ ] Create a GitHub Actions pipeline that builds and pushes Docker image
- [ ] Add deploy step to ECS or EC2
