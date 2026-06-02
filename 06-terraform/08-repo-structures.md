# Terraform & Terragrunt — Repo Structures 📁

Real-world project structures from simple to enterprise.

---

## Structure 1: Simple (Single Account, Small Team)

Best for: Learning, small projects, single AWS account.

```
my-infra/
├── main.tf              ← All resources
├── variables.tf         ← Input variables
├── outputs.tf           ← Output values
├── terraform.tfvars     ← Variable values
├── provider.tf          ← AWS provider config
├── backend.tf           ← Remote state config
└── .gitignore
```

**Pros:** Simple, easy to understand.
**Cons:** Everything in one state, risky for large projects.

---

## Structure 2: Modular (Single Account, Multiple Environments)

Best for: Small-medium projects with dev/staging/prod.

```
my-infra/
├── modules/                    ← Reusable modules
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── ecs/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── rds/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── environments/
│   ├── dev/
│   │   ├── main.tf            ← Calls modules with dev values
│   │   ├── backend.tf
│   │   └── dev.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   └── staging.tfvars
│   └── prod/
│       ├── main.tf
│       ├── backend.tf
│       └── prod.tfvars
│
└── .gitignore
```

**Usage:**
```bash
cd environments/dev
terraform init
terraform plan -var-file=dev.tfvars
terraform apply -var-file=dev.tfvars
```

**Pros:** Reusable modules, separate state per environment.
**Cons:** Still some repetition (backend.tf, provider.tf).

---

## Structure 3: Component-Based (Multiple Accounts)

Best for: Medium-large projects. Each component has its own state.

```
my-infra/
├── modules/
│   ├── vpc/
│   ├── ecs/
│   ├── rds/
│   ├── monitoring/
│   └── iam/
│
├── environments/
│   ├── dev/
│   │   ├── vpc/
│   │   │   ├── main.tf       ← Only VPC resources
│   │   │   ├── backend.tf    ← State: dev/vpc/terraform.tfstate
│   │   │   └── terraform.tfvars
│   │   ├── ecs/
│   │   │   ├── main.tf       ← Only ECS resources
│   │   │   ├── backend.tf    ← State: dev/ecs/terraform.tfstate
│   │   │   └── terraform.tfvars
│   │   └── rds/
│   │       ├── main.tf
│   │       ├── backend.tf
│   │       └── terraform.tfvars
│   ├── staging/
│   │   ├── vpc/
│   │   ├── ecs/
│   │   └── rds/
│   └── prod/
│       ├── vpc/
│       ├── ecs/
│       └── rds/
│
└── .gitignore
```

**Why separate state per component?**
- VPC rarely changes → don't risk it when deploying ECS
- RDS is critical → separate state, separate apply
- Blast radius is smaller
- Different teams can own different components

**Pros:** Small blast radius, independent deploys.
**Cons:** Lots of repeated files, cross-component references are manual.

---

## Structure 4: Terragrunt (Recommended for Real Projects)

Best for: Multiple accounts, multiple environments, teams.

```
infra-live/                          ← This repo: environment configs
├── terragrunt.hcl                   ← Root: backend + provider (defined ONCE)
│
├── dev/
│   ├── env.hcl                      ← environment = "dev", account_id = "111..."
│   ├── vpc/
│   │   └── terragrunt.hcl           ← source + inputs (5 lines)
│   ├── ecs/
│   │   └── terragrunt.hcl
│   ├── rds/
│   │   └── terragrunt.hcl
│   └── monitoring/
│       └── terragrunt.hcl
│
├── staging/
│   ├── env.hcl                      ← environment = "staging"
│   ├── vpc/
│   │   └── terragrunt.hcl
│   ├── ecs/
│   │   └── terragrunt.hcl
│   └── rds/
│       └── terragrunt.hcl
│
└── prod/
    ├── env.hcl                      ← environment = "prod"
    ├── vpc/
    │   └── terragrunt.hcl
    ├── ecs/
    │   └── terragrunt.hcl
    └── rds/
        └── terragrunt.hcl

infra-modules/                       ← Separate repo: reusable modules
├── vpc/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── ecs/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── rds/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

**Two repos:**
- `infra-live` — Environment-specific configs (what to deploy where)
- `infra-modules` — Reusable Terraform modules (how to build things)

**Why two repos?**
- Modules are versioned independently (v1.0, v1.1, v2.0)
- Dev can use module v2.0 while prod stays on v1.1
- Module changes go through PR review before any environment uses them

**Usage:**
```bash
# Deploy single component
cd infra-live/dev/vpc
terragrunt apply

# Deploy everything in dev
cd infra-live/dev
terragrunt run-all apply

# Plan everything in prod (review before applying!)
cd infra-live/prod
terragrunt run-all plan
```

---

## Structure 5: Enterprise (Multi-Account, Multi-Region)

Best for: Large organizations with AWS Organizations.

```
infra-live/
├── terragrunt.hcl                    ← Global root config
│
├── _global/                          ← Account-level resources (IAM, Organizations)
│   ├── iam/
│   │   └── terragrunt.hcl
│   └── organizations/
│       └── terragrunt.hcl
│
├── shared-services/                  ← Shared account (CI/CD, monitoring)
│   ├── account.hcl                   ← account_id, account_name
│   ├── ca-central-1/
│   │   ├── region.hcl                ← region = "ca-central-1"
│   │   ├── vpc/
│   │   │   └── terragrunt.hcl
│   │   ├── jenkins/
│   │   │   └── terragrunt.hcl
│   │   └── monitoring/
│   │       └── terragrunt.hcl
│   └── us-east-1/
│       ├── region.hcl
│       └── cloudfront/
│           └── terragrunt.hcl
│
├── dev/
│   ├── account.hcl
│   ├── ca-central-1/
│   │   ├── region.hcl
│   │   ├── vpc/
│   │   │   └── terragrunt.hcl
│   │   ├── ecs/
│   │   │   └── terragrunt.hcl
│   │   └── rds/
│   │       └── terragrunt.hcl
│   └── us-east-1/
│       └── ...
│
├── staging/
│   ├── account.hcl
│   └── ca-central-1/
│       └── ...
│
└── prod/
    ├── account.hcl
    ├── ca-central-1/
    │   ├── region.hcl
    │   ├── vpc/
    │   ├── ecs/
    │   ├── rds/
    │   └── elasticache/
    └── us-east-1/
        ├── region.hcl
        └── dr/                       ← Disaster recovery
            └── terragrunt.hcl
```

**Config hierarchy:**
```
terragrunt.hcl (root)     → Backend, common tags
  └── account.hcl          → Account ID, account name
      └── region.hcl       → Region
          └── terragrunt.hcl → Component-specific inputs
```

**Root terragrunt.hcl for enterprise:**
```hcl
locals {
  account = read_terragrunt_config(find_in_parent_folders("account.hcl"))
  region  = read_terragrunt_config(find_in_parent_folders("region.hcl"))
}

remote_state {
  backend = "s3"
  config = {
    bucket         = "company-terraform-state-${local.account.locals.account_id}"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = local.region.locals.region
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }
}

generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite_terragrunt"
  contents  = <<EOF
provider "aws" {
  region = "${local.region.locals.region}"

  assume_role {
    role_arn = "arn:aws:iam::${local.account.locals.account_id}:role/TerraformRole"
  }

  default_tags {
    tags = {
      ManagedBy   = "terraform"
      Account     = "${local.account.locals.account_name}"
      Region      = "${local.region.locals.region}"
    }
  }
}
EOF
}
```

---

## Which Structure Should You Use?

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Learning / Personal project?                            │
│    → Structure 1 (Simple)                                │
│                                                          │
│  Small team, 1 account, 2-3 environments?                │
│    → Structure 2 (Modular)                               │
│                                                          │
│  Multiple components, want separate state?               │
│    → Structure 3 (Component-based)                       │
│                                                          │
│  Multiple accounts, tired of copy-paste?                 │
│    → Structure 4 (Terragrunt) ← MOST COMMON IN JOBS     │
│                                                          │
│  Large org, multi-account, multi-region?                 │
│    → Structure 5 (Enterprise Terragrunt)                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## State File Organization

No matter which structure, your S3 state bucket should mirror your folder structure:

```
s3://my-terraform-state/
├── dev/
│   ├── vpc/terraform.tfstate
│   ├── ecs/terraform.tfstate
│   └── rds/terraform.tfstate
├── staging/
│   ├── vpc/terraform.tfstate
│   ├── ecs/terraform.tfstate
│   └── rds/terraform.tfstate
└── prod/
    ├── vpc/terraform.tfstate
    ├── ecs/terraform.tfstate
    └── rds/terraform.tfstate
```

Terragrunt does this automatically with `path_relative_to_include()`.

---

## Module Versioning (infra-modules repo)

```hcl
# In infra-live, pin module versions
terraform {
  source = "git::git@github.com:myorg/infra-modules.git//vpc?ref=v1.2.0"
}

# Dev can test new version
terraform {
  source = "git::git@github.com:myorg/infra-modules.git//vpc?ref=v2.0.0"
}

# Prod stays on stable version
terraform {
  source = "git::git@github.com:myorg/infra-modules.git//vpc?ref=v1.2.0"
}
```

**Workflow:**
1. Developer updates module in `infra-modules` repo
2. PR reviewed and merged → tagged as `v2.0.0`
3. Update `infra-live/dev` to use `v2.0.0` → test
4. Update `infra-live/staging` → test more
5. Update `infra-live/prod` → deploy with confidence

---

*Start simple, grow as needed. Don't over-engineer on day one! 📁*
