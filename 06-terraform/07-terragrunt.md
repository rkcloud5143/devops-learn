# Terragrunt — Why, What & How 🌿

Terraform is great. Terragrunt makes it better for real-world projects.

---

## The Problem Terragrunt Solves

### Without Terragrunt (Pure Terraform)

```
environments/
├── dev/
│   ├── vpc/
│   │   ├── main.tf          ← Copy of module call
│   │   ├── backend.tf       ← backend "s3" { bucket = "state" key = "dev/vpc" }
│   │   ├── provider.tf      ← provider "aws" { region = "ca-central-1" }
│   │   └── terraform.tfvars
│   ├── ecs/
│   │   ├── main.tf          ← Copy of module call
│   │   ├── backend.tf       ← backend "s3" { bucket = "state" key = "dev/ecs" }  ← REPEATED!
│   │   ├── provider.tf      ← provider "aws" { region = "ca-central-1" }         ← REPEATED!
│   │   └── terraform.tfvars
│   └── rds/
│       ├── main.tf
│       ├── backend.tf       ← REPEATED AGAIN!
│       ├── provider.tf      ← REPEATED AGAIN!
│       └── terraform.tfvars
├── staging/
│   ├── vpc/                 ← Same structure, different values
│   ├── ecs/                 ← EVERYTHING REPEATED!
│   └── rds/
└── prod/
    ├── vpc/
    ├── ecs/
    └── rds/
```

**Problems:**
- `backend.tf` repeated in every folder (only the key changes)
- `provider.tf` repeated everywhere
- Change the state bucket? Update 9+ files
- Change the region? Update 9+ files
- Lots of copy-paste, easy to make mistakes

### With Terragrunt (DRY — Don't Repeat Yourself)

```
environments/
├── terragrunt.hcl           ← ROOT: backend + provider defined ONCE
├── dev/
│   ├── env.hcl              ← environment = "dev"
│   ├── vpc/
│   │   └── terragrunt.hcl   ← Just: source + inputs (3-5 lines)
│   ├── ecs/
│   │   └── terragrunt.hcl
│   └── rds/
│       └── terragrunt.hcl
├── staging/
│   ├── env.hcl              ← environment = "staging"
│   ├── vpc/
│   │   └── terragrunt.hcl
│   ├── ecs/
│   │   └── terragrunt.hcl
│   └── rds/
│       └── terragrunt.hcl
└── prod/
    ├── env.hcl
    ├── vpc/
    │   └── terragrunt.hcl
    ├── ecs/
    │   └── terragrunt.hcl
    └── rds/
        └── terragrunt.hcl

modules/                      ← Reusable Terraform modules
├── vpc/
├── ecs/
└── rds/
```

**What changed:**
- Backend config defined **once** in root `terragrunt.hcl`
- Provider config defined **once**
- Each component is just a tiny `terragrunt.hcl` with source + inputs
- Change state bucket? Update **1 file**

---

## How Terragrunt Works

### Root terragrunt.hcl (Define Once)

```hcl
# environments/terragrunt.hcl

# Generate backend config for ALL child modules
remote_state {
  backend = "s3"
  config = {
    bucket         = "my-company-terraform-state"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "ca-central-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
  # Auto-create the S3 bucket and DynamoDB table if they don't exist!
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }
}

# Generate provider config for ALL child modules
generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite_terragrunt"
  contents  = <<EOF
provider "aws" {
  region = "ca-central-1"

  default_tags {
    tags = {
      ManagedBy = "terraform"
    }
  }
}
EOF
}
```

### Environment-level config

```hcl
# environments/dev/env.hcl
locals {
  environment = "dev"
  account_id  = "111111111111"
}
```

```hcl
# environments/prod/env.hcl
locals {
  environment = "prod"
  account_id  = "222222222222"
}
```

### Child terragrunt.hcl (Tiny!)

```hcl
# environments/dev/vpc/terragrunt.hcl

# Include root config (backend + provider)
include "root" {
  path = find_in_parent_folders()
}

# Read environment variables
locals {
  env = read_terragrunt_config(find_in_parent_folders("env.hcl"))
}

# Point to the reusable module
terraform {
  source = "../../../modules/vpc"
}

# Pass inputs to the module
inputs = {
  environment    = local.env.locals.environment
  vpc_cidr       = "10.0.0.0/16"
  public_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
}
```

That's it! **No backend.tf, no provider.tf** — Terragrunt generates them.

---

## Terragrunt Dependencies

When one module depends on another (ECS needs VPC ID):

```hcl
# environments/dev/ecs/terragrunt.hcl

include "root" {
  path = find_in_parent_folders()
}

# Declare dependency on VPC
dependency "vpc" {
  config_path = "../vpc"
}

terraform {
  source = "../../../modules/ecs"
}

inputs = {
  vpc_id     = dependency.vpc.outputs.vpc_id
  subnet_ids = dependency.vpc.outputs.private_subnet_ids
}
```

Terragrunt automatically:
- Runs VPC first
- Reads VPC outputs
- Passes them to ECS

---

## Terragrunt Commands

```bash
# Same as Terraform, just replace "terraform" with "terragrunt"
terragrunt init
terragrunt plan
terragrunt apply
terragrunt destroy

# Run across ALL modules in a directory
terragrunt run-all plan      # Plan everything
terragrunt run-all apply     # Apply everything (respects dependencies)
terragrunt run-all destroy   # Destroy everything (reverse order)
```

### run-all is Powerful

```bash
cd environments/dev
terragrunt run-all apply

# Terragrunt figures out the order:
# 1. VPC (no dependencies)
# 2. RDS + ECS (depend on VPC, run in parallel)
# 3. App (depends on ECS + RDS)
```

---

## Terragrunt with IAM Roles

### Per-Environment Roles

```hcl
# environments/terragrunt.hcl (root)

locals {
  env = read_terragrunt_config(find_in_parent_folders("env.hcl"))
}

generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite_terragrunt"
  contents  = <<EOF
provider "aws" {
  region = "ca-central-1"

  assume_role {
    role_arn = "arn:aws:iam::${local.env.locals.account_id}:role/TerraformRole"
  }
}
EOF
}
```

Now:
- `dev/` → assumes role in dev account (111111111111)
- `prod/` → assumes role in prod account (222222222222)
- **One set of credentials, multiple accounts!**

---

## Terraform vs Terragrunt — When to Use What

```
┌─────────────────────────────────────────────────────────────┐
│                    DECISION TREE                            │
│                                                             │
│  Small project, 1 environment, 1 person?                    │
│    → Plain Terraform is fine                                │
│                                                             │
│  Multiple environments (dev/staging/prod)?                  │
│    → Terragrunt saves you from copy-paste                   │
│                                                             │
│  Multiple AWS accounts?                                     │
│    → Terragrunt with assume_role per environment            │
│                                                             │
│  Team of engineers?                                         │
│    → Terragrunt keeps things consistent                     │
│                                                             │
│  Lots of modules with dependencies?                         │
│    → Terragrunt run-all handles ordering                    │
└─────────────────────────────────────────────────────────────┘
```

| Feature | Terraform | Terragrunt |
|---------|-----------|------------|
| Create resources | ✅ | ✅ (wraps Terraform) |
| Remote state | Manual setup | Auto-creates bucket + table |
| DRY backend config | ❌ Repeat per folder | ✅ Define once |
| DRY provider config | ❌ Repeat per folder | ✅ Define once |
| Cross-module dependencies | Manual (remote_state data) | Built-in (dependency block) |
| Apply all modules | Manual, one by one | `run-all apply` |
| Destroy in order | Manual, reverse order | `run-all destroy` |

---

*Think of Terragrunt as the manager that keeps all your Terraform modules organized! 🌿*
