# Terraform — Command Reference 🏗️

Every Terraform command you'll use, with examples.

---

## Core Workflow

```bash
# ─── The 4 commands you use 90% of the time ───

terraform init                         # Initialize (download providers, modules)
terraform plan                         # Preview changes (dry run)
terraform apply                        # Apply changes (create/update resources)
terraform destroy                      # Destroy all managed resources
```

---

## Init

```bash
terraform init                         # Standard init
terraform init -upgrade                # Upgrade providers to latest allowed
terraform init -reconfigure            # Reconfigure backend (ignore existing)
terraform init -migrate-state          # Migrate state to new backend
terraform init -backend=false          # Skip backend config (for validation only)
terraform init -get=false              # Skip module download
```

---

## Plan

```bash
terraform plan                         # Standard plan
terraform plan -out=plan.tfplan        # Save plan to file (for apply)
terraform plan -var="env=prod"         # Pass variable
terraform plan -var-file=prod.tfvars   # Use variable file
terraform plan -target=aws_instance.web  # Plan specific resource only
terraform plan -destroy                # Preview destroy
terraform plan -refresh-only           # Only refresh state, no changes
terraform plan -json                   # JSON output (for CI/CD)
terraform plan -no-color               # No color (for logs)
terraform plan -parallelism=20         # Parallel operations (default: 10)
```

---

## Apply

```bash
terraform apply                        # Apply (prompts for confirmation)
terraform apply -auto-approve          # Skip confirmation (CI/CD)
terraform apply plan.tfplan            # Apply saved plan
terraform apply -var="env=prod"        # Pass variable
terraform apply -var-file=prod.tfvars  # Use variable file
terraform apply -target=aws_instance.web  # Apply specific resource
terraform apply -replace=aws_instance.web  # Force recreate resource
terraform apply -parallelism=20        # Parallel operations
terraform apply -lock=false            # Skip state locking (dangerous)
terraform apply -lock-timeout=5m       # Wait for lock
```

---

## Destroy

```bash
terraform destroy                      # Destroy everything
terraform destroy -auto-approve        # Skip confirmation
terraform destroy -target=aws_instance.web  # Destroy specific resource
terraform plan -destroy                # Preview what will be destroyed
```

---

## State Management

```bash
# ─── View State ───
terraform state list                   # List all resources in state
terraform state show aws_instance.web  # Show resource details
terraform show                         # Show entire state
terraform show -json                   # JSON format

# ─── Modify State ───
terraform state mv aws_instance.old aws_instance.new  # Rename resource
terraform state mv 'aws_instance.web' 'module.compute.aws_instance.web'  # Move to module
terraform state rm aws_instance.web    # Remove from state (resource stays in AWS)
terraform state replace-provider hashicorp/aws registry.example.com/aws  # Change provider

# ─── Remote State ───
terraform state pull                   # Download remote state to stdout
terraform state pull > state.json      # Save to file
terraform state push state.json        # Upload state (DANGEROUS)

# ─── Import ───
terraform import aws_instance.web i-1234567890abcdef0  # Import existing resource
terraform import aws_s3_bucket.data my-bucket-name
terraform import 'aws_security_group.web' sg-0123456789abcdef0
terraform import 'module.vpc.aws_vpc.main' vpc-0123456789abcdef0
```

---

## Workspace

```bash
terraform workspace list               # List workspaces
terraform workspace show               # Current workspace
terraform workspace new dev            # Create workspace
terraform workspace select dev         # Switch workspace
terraform workspace delete dev         # Delete workspace

# Use in config
# terraform.workspace returns current workspace name
# resource "aws_instance" "web" {
#   tags = { Environment = terraform.workspace }
# }
```

---

## Format & Validate

```bash
terraform fmt                          # Format all .tf files
terraform fmt -check                   # Check formatting (CI/CD)
terraform fmt -diff                    # Show formatting changes
terraform fmt -recursive               # Format subdirectories too

terraform validate                     # Validate syntax and config
terraform validate -json               # JSON output
```

---

## Output

```bash
terraform output                       # Show all outputs
terraform output vpc_id                # Show specific output
terraform output -json                 # JSON format
terraform output -raw vpc_id           # Raw value (no quotes)
```

---

## Graph & Debug

```bash
terraform graph                        # Generate dependency graph (DOT format)
terraform graph | dot -Tpng > graph.png  # Render to image (needs graphviz)

terraform providers                    # List providers used
terraform providers lock               # Generate lock file
terraform version                      # Terraform version

# Debug
TF_LOG=DEBUG terraform plan            # Enable debug logging
TF_LOG=TRACE terraform apply           # Maximum verbosity
TF_LOG_PATH=terraform.log terraform plan  # Log to file
```

---

## Provider & Module

```bash
# Providers
terraform providers                    # List required providers
terraform providers lock               # Update lock file
terraform providers mirror /path       # Mirror providers locally

# Modules
terraform get                          # Download modules
terraform get -update                  # Update modules
```

---

## Console (Interactive)

```bash
terraform console                      # Start interactive console

# Inside console:
# > var.environment
# "production"
# > length(var.subnets)
# 3
# > cidrsubnet("10.0.0.0/16", 8, 1)
# "10.0.1.0/24"
# > exit
```

---

## CI/CD Patterns

```bash
# ─── GitHub Actions / Jenkins ───

# Plan on PR
terraform init -input=false
terraform plan -input=false -no-color -out=plan.tfplan

# Apply on merge to main
terraform init -input=false
terraform apply -input=false -auto-approve plan.tfplan

# ─── With specific backend ───
terraform init \
  -backend-config="bucket=my-state" \
  -backend-config="key=prod/terraform.tfstate" \
  -backend-config="region=ca-central-1"

# ─── Destroy in CI (cleanup) ───
terraform destroy -auto-approve -input=false
```

---

## Common Flags Reference

| Flag | Description |
|------|-------------|
| `-auto-approve` | Skip confirmation prompt |
| `-var="key=value"` | Set variable |
| `-var-file=file.tfvars` | Use variable file |
| `-target=resource` | Target specific resource |
| `-replace=resource` | Force recreate resource |
| `-out=file` | Save plan to file |
| `-input=false` | Disable interactive prompts |
| `-no-color` | Disable color output |
| `-json` | JSON output |
| `-parallelism=N` | Concurrent operations |
| `-lock=false` | Disable state locking |
| `-lock-timeout=Ns` | Wait for state lock |
| `-refresh=false` | Skip state refresh |
| `-compact-warnings` | Show warnings compactly |

---

## .terraform.lock.hcl

```bash
# Lock file — pins provider versions
# Commit to Git for consistent builds

terraform providers lock               # Generate/update lock file
terraform providers lock -platform=linux_amd64  # Lock for specific platform
terraform init -upgrade                # Upgrade providers and update lock
```

---

*Print this and keep it next to your terminal! 🏗️*
