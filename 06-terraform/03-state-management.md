# Terraform — State Management

```
┌─── Local State (default) ──────┐    ┌─── Remote State (teams) ──────┐
│                                 │    │                                │
│  terraform.tfstate on your      │    │  terraform.tfstate in S3       │
│  local machine                  │    │  + DynamoDB lock table         │
│                                 │    │                                │
│  Problem: only you can use it   │    │  ✓ Team can share state       │
│  Problem: can lose it           │    │  ✓ Locking prevents conflicts │
│                                 │    │  ✓ Versioned in S3            │
└─────────────────────────────────┘    └────────────────────────────────┘
```

## Remote Backend Config
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "dev/terraform.tfstate"
    region         = "ca-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

## Checklist
- [ ] Set up remote state in S3 + DynamoDB
