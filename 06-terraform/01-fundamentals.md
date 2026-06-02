# Terraform — Fundamentals

## Why Terraform?
```
Manual (Console):                  Terraform:
┌──────────────────┐              ┌──────────────────┐
│ Click, click,    │              │ Write code once   │
│ click in AWS     │              │ Run: terraform    │
│ console...       │              │      apply        │
│                  │              │                   │
│ - Not repeatable │              │ - Repeatable      │
│ - No history     │              │ - Version control │
│ - Error prone    │              │ - Review changes  │
│ - Can't review   │              │ - Destroy easily  │
└──────────────────┘              └──────────────────┘
```

## Workflow
```
Write .tf files
      │
      ▼
terraform init       ← Download providers (AWS, Azure, etc.)
      │
      ▼
terraform plan       ← Preview what will change (dry run)
      │
      ▼
terraform apply      ← Create/update real infrastructure
      │
      ▼
terraform destroy    ← Tear everything down (cleanup)

State File (terraform.tfstate):
- Tracks what Terraform created
- NEVER edit manually
- Store in S3 bucket for team use (remote backend)
```

## Checklist
- [ ] Install Terraform
