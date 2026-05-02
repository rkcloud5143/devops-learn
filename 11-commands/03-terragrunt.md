# Terragrunt — Command Reference 🌿

Every Terragrunt command, plus the extras it adds on top of Terraform.

---

## Core Workflow (Same as Terraform)

```bash
# Terragrunt wraps Terraform — same commands, just replace "terraform" with "terragrunt"

terragrunt init                        # Initialize
terragrunt plan                        # Preview changes
terragrunt apply                       # Apply changes
terragrunt destroy                     # Destroy resources
terragrunt output                      # Show outputs
terragrunt validate                    # Validate config
terragrunt fmt                         # Format HCL files
terragrunt console                     # Interactive console
```

All Terraform flags work with Terragrunt:
```bash
terragrunt plan -var="env=prod"
terragrunt apply -auto-approve
terragrunt apply -target=aws_instance.web
terragrunt destroy -auto-approve
terragrunt plan -out=plan.tfplan
```

---

## run-all (The Killer Feature)

Run commands across ALL modules in a directory tree. Terragrunt handles dependency ordering.

```bash
# ─── Plan everything ───
terragrunt run-all plan

# ─── Apply everything (respects dependencies) ───
terragrunt run-all apply

# ─── Destroy everything (reverse dependency order) ───
terragrunt run-all destroy

# ─── Init all modules ───
terragrunt run-all init

# ─── Validate all modules ───
terragrunt run-all validate

# ─── Output from all modules ───
terragrunt run-all output
```

### run-all Flags

```bash
# Skip confirmation
terragrunt run-all apply --terragrunt-non-interactive

# Exclude specific modules
terragrunt run-all plan --terragrunt-exclude-dir="*/rds"

# Include only specific modules
terragrunt run-all plan --terragrunt-include-dir="*/vpc" --terragrunt-include-dir="*/ecs"

# Ignore dependency errors (continue even if dependency fails)
terragrunt run-all apply --terragrunt-ignore-dependency-errors

# Ignore external dependencies
terragrunt run-all plan --terragrunt-ignore-external-dependencies

# Limit parallelism
terragrunt run-all apply --terragrunt-parallelism 3
```

### run-all Example

```bash
# Directory structure:
# environments/dev/
# ├── vpc/terragrunt.hcl
# ├── ecs/terragrunt.hcl      (depends on vpc)
# ├── rds/terragrunt.hcl      (depends on vpc)
# └── app/terragrunt.hcl      (depends on ecs + rds)

cd environments/dev
terragrunt run-all apply

# Terragrunt figures out the order:
# 1. vpc          (no dependencies)
# 2. ecs + rds    (depend on vpc, run in PARALLEL)
# 3. app          (depends on ecs + rds, runs last)
```

---

## Terragrunt-Specific Flags

```bash
# ─── General ───
--terragrunt-config FILE               # Use specific config file (default: terragrunt.hcl)
--terragrunt-working-dir DIR           # Set working directory
--terragrunt-non-interactive           # Skip all prompts
--terragrunt-log-level LEVEL           # debug, info, warn, error
--terragrunt-no-color                  # Disable color output

# ─── Source ───
--terragrunt-source PATH               # Override terraform source
--terragrunt-source-update             # Force re-download of source
--terragrunt-source-map                # Map source URLs

# ─── Dependencies ───
--terragrunt-ignore-dependency-errors  # Continue on dependency failure
--terragrunt-ignore-external-dependencies  # Ignore deps outside working dir
--terragrunt-include-external-dependencies # Include external deps in run-all

# ─── Directories ───
--terragrunt-exclude-dir DIR           # Exclude directory from run-all
--terragrunt-include-dir DIR           # Include only this directory
--terragrunt-strict-include            # Only include explicitly included dirs

# ─── Cache ───
--terragrunt-download-dir DIR          # Where to download Terraform code
--terragrunt-no-auto-init              # Don't auto-run init
--terragrunt-no-auto-retry             # Don't auto-retry on errors

# ─── Parallelism ───
--terragrunt-parallelism N             # Max parallel operations in run-all
```

---

## Debugging

```bash
# Show what Terragrunt is doing
TERRAGRUNT_LOG_LEVEL=debug terragrunt plan

# Show the generated Terraform files
terragrunt plan
ls -la .terragrunt-cache/              # Generated files are here

# Show dependency graph
terragrunt graph-dependencies          # Print dependency graph
terragrunt graph-dependencies | dot -Tpng > deps.png  # Render image

# Show rendered terragrunt.hcl
terragrunt render-json                 # Show resolved config as JSON

# Show info about the module
terragrunt terragrunt-info             # Working dir, download dir, etc.

# Validate HCL syntax
terragrunt hclfmt                      # Format terragrunt.hcl files
terragrunt hclfmt --check              # Check formatting
```

---

## State Commands (via Terragrunt)

```bash
# All terraform state commands work through terragrunt
terragrunt state list
terragrunt state show aws_instance.web
terragrunt state mv aws_instance.old aws_instance.new
terragrunt state rm aws_instance.web
terragrunt state pull
terragrunt import aws_instance.web i-1234567890
```

---

## Common terragrunt.hcl Functions

```hcl
# ─── Path Functions ───
find_in_parent_folders()               # Find root terragrunt.hcl
find_in_parent_folders("env.hcl")      # Find specific file in parents
get_terragrunt_dir()                   # Current terragrunt.hcl directory
get_parent_terragrunt_dir()            # Parent terragrunt.hcl directory
get_original_terragrunt_dir()          # Original working directory
path_relative_to_include()             # Relative path from root to current
path_relative_from_include()           # Relative path from current to root

# ─── Read Functions ───
read_terragrunt_config("env.hcl")      # Read another terragrunt config
read_terragrunt_config(find_in_parent_folders("env.hcl"))

# ─── AWS Functions ───
get_aws_account_id()                   # Current AWS account ID
get_aws_caller_identity_arn()          # Current caller ARN
get_aws_caller_identity_user_id()      # Current user ID

# ─── Terraform Functions ───
run_cmd("aws", "sts", "get-caller-identity")  # Run shell command
```

---

## CI/CD Patterns

```bash
# ─── Plan all in PR ───
cd environments/dev
terragrunt run-all plan \
  --terragrunt-non-interactive \
  --terragrunt-no-color \
  2>&1 | tee plan-output.txt

# ─── Apply all on merge ───
cd environments/dev
terragrunt run-all apply \
  --terragrunt-non-interactive \
  --terragrunt-parallelism 3

# ─── Apply single module ───
cd environments/dev/vpc
terragrunt apply -auto-approve

# ─── Destroy specific environment ───
cd environments/dev
terragrunt run-all destroy \
  --terragrunt-non-interactive

# ─── Override source for testing ───
cd environments/dev/vpc
terragrunt plan --terragrunt-source /local/path/to/module
```

---

## Scaffold New Module

```bash
# Quick way to add a new component
mkdir -p environments/dev/new-service
cat > environments/dev/new-service/terragrunt.hcl << 'EOF'
include "root" {
  path = find_in_parent_folders()
}

locals {
  env = read_terragrunt_config(find_in_parent_folders("env.hcl"))
}

terraform {
  source = "../../../modules/new-service"
}

inputs = {
  environment = local.env.locals.environment
  # Add your inputs here
}
EOF
```

---

## Terragrunt vs Terraform Commands

| Task | Terraform | Terragrunt |
|------|-----------|------------|
| Init | `terraform init` | `terragrunt init` |
| Plan | `terraform plan` | `terragrunt plan` |
| Apply | `terraform apply` | `terragrunt apply` |
| Destroy | `terraform destroy` | `terragrunt destroy` |
| Plan ALL modules | Manual, one by one | `terragrunt run-all plan` |
| Apply ALL modules | Manual, one by one | `terragrunt run-all apply` |
| Dependency graph | `terraform graph` | `terragrunt graph-dependencies` |
| Format | `terraform fmt` | `terragrunt hclfmt` |
| Override source | N/A | `--terragrunt-source` |

---

*Terragrunt = Terraform commands + run-all + DRY config! 🌿*
