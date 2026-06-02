# Interview Prep — Troubleshooting Scenarios

## 1. EC2 instance can't reach the internet — what do you check?
```
Checklist:
├── Is it in a public subnet?
├── Does it have a public IP or Elastic IP?
├── Is there an Internet Gateway attached to VPC?
├── Does the route table have 0.0.0.0/0 → IGW?
├── Does the Security Group allow outbound traffic?
└── Does the NACL allow outbound + inbound?
```

## 2. Application is slow — how do you diagnose?
```
Checklist:
├── Check CloudWatch metrics (CPU, memory, network)
├── Check application logs
├── Check database performance (RDS metrics)
├── Check ALB response times
├── Check if Auto Scaling is hitting limits
└── Check for network bottlenecks
```

## 3. Deployment failed — what's your rollback strategy?
```
├── ECS: previous task definition is still available
├── Kubernetes: kubectl rollout undo
├── Terraform: git revert + terraform apply
└── Lambda: publish versions, alias to previous
```
