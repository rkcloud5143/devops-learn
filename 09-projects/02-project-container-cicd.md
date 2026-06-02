# Project 2: Container CI/CD Pipeline

## Architecture
```
Developer                GitHub              AWS
   │                       │                  │
   │  git push             │                  │
   ├──────────────────────►│                  │
   │                       │                  │
   │              GitHub Actions triggers     │
   │                       │                  │
   │               ┌───────▼────────┐         │
   │               │  Build Stage   │         │
   │               │  - Checkout    │         │
   │               │  - Run tests   │         │
   │               │  - Build image │         │
   │               └───────┬────────┘         │
   │                       │                  │
   │               ┌───────▼────────┐   ┌─────▼──────┐
   │               │  Push to ECR   │──►│    ECR     │
   │               └───────┬────────┘   │ (Registry) │
   │                       │            └────────────┘
   │               ┌───────▼────────┐         │
   │               │ Deploy to ECS  │         │
   │               │ Fargate        │──►┌─────▼──────┐
   │               └────────────────┘   │    ECS     │
   │                                    │  Fargate   │
   │                                    │ ┌────────┐ │
   │                                    │ │Task 1  │ │
   │                                    │ │Task 2  │ │
   │                                    │ └────────┘ │
   │                                    └────────────┘
```

## What to Build
```
Complete pipeline:
├── Simple Node.js or Python web app
├── Dockerfile (multi-stage build)
├── docker-compose.yml for local dev
├── GitHub Actions workflow:
│   ├── On PR: build + test
│   └── On merge to main: build + push to ECR + deploy to ECS
├── Terraform for ECS infrastructure:
│   ├── ECS Cluster
│   ├── ECS Service + Task Definition
│   ├── ECR Repository
│   ├── ALB for the service
│   └── CloudWatch log group
└── README with setup instructions
```
