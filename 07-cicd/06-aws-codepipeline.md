# AWS CodePipeline & CodeBuild — AWS-Native CI/CD 🔶

CI/CD without leaving the AWS ecosystem.

---

## Overview

```
┌─── AWS CI/CD Services ──────────────────────────────────────────┐
│                                                                   │
│  CodeCommit    = Git repository (like GitHub)                    │
│  CodeBuild     = Build & test (like GitHub Actions runner)       │
│  CodeDeploy    = Deploy to EC2/ECS/Lambda                        │
│  CodePipeline  = Orchestrates all of the above                   │
│                                                                   │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐    │
│  │ Source    │──▶│ Build    │──▶│ Test     │──▶│ Deploy   │    │
│  │CodeCommit│   │CodeBuild │   │CodeBuild │   │CodeDeploy│    │
│  │or GitHub │   │          │   │          │   │or ECS    │    │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘    │
│  └──────────────── CodePipeline ──────────────────────────┘     │
└──────────────────────────────────────────────────────────────────┘
```

---

## CodeBuild — buildspec.yml

```yaml
# buildspec.yml (in repo root)
version: 0.2

env:
  variables:
    AWS_REGION: ca-central-1
    ECR_REPO: 123456789012.dkr.ecr.ca-central-1.amazonaws.com/my-app
  secrets-manager:
    DB_PASSWORD: prod/db:password    # Fetch from Secrets Manager

phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm ci

  pre_build:
    commands:
      - echo "Running tests..."
      - npm test
      - echo "Logging into ECR..."
      - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO

  build:
    commands:
      - echo "Building Docker image..."
      - docker build -t $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION .
      - docker tag $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION $ECR_REPO:latest

  post_build:
    commands:
      - echo "Pushing to ECR..."
      - docker push $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION
      - docker push $ECR_REPO:latest
      - echo "Writing image definition for ECS..."
      - printf '[{"name":"my-app","imageUri":"%s"}]' $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json

cache:
  paths:
    - 'node_modules/**/*'
```

---

## CodePipeline with Terraform

```hcl
resource "aws_codepipeline" "main" {
  name     = "my-app-pipeline"
  role_arn = aws_iam_role.codepipeline.arn

  artifact_store {
    location = aws_s3_bucket.artifacts.bucket
    type     = "S3"
  }

  stage {
    name = "Source"
    action {
      name             = "GitHub"
      category         = "Source"
      owner            = "ThirdParty"
      provider         = "GitHub"
      version          = "1"
      output_artifacts = ["source"]
      configuration = {
        Owner  = "myorg"
        Repo   = "my-app"
        Branch = "main"
      }
    }
  }

  stage {
    name = "Build"
    action {
      name             = "Build"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source"]
      output_artifacts = ["build"]
      configuration = {
        ProjectName = aws_codebuild_project.main.name
      }
    }
  }

  stage {
    name = "Deploy"
    action {
      name            = "Deploy"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "ECS"
      version         = "1"
      input_artifacts = ["build"]
      configuration = {
        ClusterName = "my-cluster"
        ServiceName = "my-service"
      }
    }
  }
}
```

---

## AWS CI/CD vs GitHub Actions vs Jenkins

| Feature | CodePipeline | GitHub Actions | Jenkins |
|---------|-------------|---------------|---------|
| Hosting | AWS managed | GitHub managed | Self-hosted |
| AWS integration | Native | Via actions | Via plugins |
| Cost | Pay per pipeline | Free tier | Free (+ server cost) |
| Config | Console/Terraform | YAML in repo | Jenkinsfile |
| Flexibility | Limited | High | Highest |
| Best for | AWS-only shops | GitHub repos | Enterprise |

**Use CodePipeline** when you're all-in on AWS and want tight integration.
**Use GitHub Actions** for most teams — simpler, more flexible.

---

*AWS-native CI/CD — great when you're already deep in the AWS ecosystem! 🔶*
