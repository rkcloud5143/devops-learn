# Docker — AWS Container Services

```
┌─── Where to Run Containers on AWS ──────────────────┐
│                                                       │
│  ECR (Elastic Container Registry)                    │
│  └── Store your Docker images (like Docker Hub)      │
│                                                       │
│  ECS (Elastic Container Service)                     │
│  └── Run containers (AWS-managed orchestration)      │
│      ├── EC2 launch type: you manage servers         │
│      └── Fargate: serverless (no servers to manage)  │
│                                                       │
│  EKS (Elastic Kubernetes Service)                    │
│  └── Run Kubernetes on AWS (more complex, more power)│
│                                                       │
│  For beginners: ECR + ECS Fargate                    │
│  For interviews: know both ECS and EKS               │
└───────────────────────────────────────────────────────┘
```

## Checklist
- [ ] Push an image to Docker Hub
- [ ] Push an image to AWS ECR
