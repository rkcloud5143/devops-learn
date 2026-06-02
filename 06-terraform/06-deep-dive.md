# Terraform — Deep Dive 🏗️

Everything you need to know beyond the basics.

---

## How Terraform Actually Works

```
                    You write .tf files
                          │
                          ▼
              ┌───────────────────────┐
              │    terraform init     │
              │  Downloads providers  │
              │  (AWS, Azure, etc.)   │
              └───────┬───────────────┘
                      │
                      ▼
              ┌───────────────────────┐
              │    terraform plan     │
              │  Reads state file     │──→ Compares with real AWS
              │  Shows what changes   │
              └───────┬───────────────┘
                      │
                      ▼
              ┌───────────────────────┐
              │    terraform apply    │
              │  Makes API calls to   │──→ Creates/updates resources
              │  AWS (or any cloud)   │
              └───────┬───────────────┘
                      │
                      ▼
              ┌───────────────────────┐
              │   State file updated  │
              │  terraform.tfstate    │──→ Records what was created
              └───────────────────────┘
```

---

## Modules — Reusable Building Blocks

### What is a Module?
A folder with `.tf` files that you can reuse. Like a function in programming.

### Without Modules (Copy-Paste Mess)
```
# You copy-paste the same VPC code for dev, staging, prod
# Change one thing? Update 3 places. Miss one? Bug.
```

### With Modules (Clean & Reusable)
```hcl
# modules/vpc/main.tf
resource "aws_vpc" "this" {
  cidr_block = var.cidr_block
  tags       = { Name = var.name }
}

resource "aws_subnet" "public" {
  count      = length(var.public_subnets)
  vpc_id     = aws_vpc.this.id
  cidr_block = var.public_subnets[count.index]
}

# modules/vpc/variables.tf
variable "cidr_block" {}
variable "name" {}
variable "public_subnets" { type = list(string) }

# modules/vpc/outputs.tf
output "vpc_id" { value = aws_vpc.this.id }
```

### Using the Module
```hcl
# environments/dev/main.tf
module "vpc" {
  source         = "../../modules/vpc"
  cidr_block     = "10.0.0.0/16"
  name           = "dev-vpc"
  public_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
}

# environments/prod/main.tf
module "vpc" {
  source         = "../../modules/vpc"
  cidr_block     = "10.1.0.0/16"
  name           = "prod-vpc"
  public_subnets = ["10.1.1.0/24", "10.1.2.0/24"]
}
```

**Pizza shop:** A module is a recipe. Write it once, use it in every kitchen.

---

## Data Sources — Read Existing Resources

```hcl
# Find the latest Amazon Linux AMI (don't hardcode AMI IDs!)
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Use it
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
}

# Look up existing VPC
data "aws_vpc" "existing" {
  tags = { Name = "my-existing-vpc" }
}
```

---

## Lifecycle Rules

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t3.micro"

  lifecycle {
    # Create new one before destroying old (zero downtime)
    create_before_destroy = true

    # Prevent accidental deletion
    prevent_destroy = true

    # Ignore changes made outside Terraform
    ignore_changes = [tags, ami]
  }
}
```

---

## Dynamic Blocks

When you need a variable number of nested blocks:

```hcl
variable "ingress_rules" {
  default = [
    { port = 80, description = "HTTP" },
    { port = 443, description = "HTTPS" },
    { port = 22, description = "SSH" },
  ]
}

resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = ingress.value.description
    }
  }
}
```

---

## Workspaces vs Directories

Two ways to manage environments:

### Workspaces (Simple but Limited)
```bash
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
terraform apply
```
```hcl
# Same code, different state per workspace
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
  tags = { Environment = terraform.workspace }
}
```

### Separate Directories (Recommended)
```
environments/
├── dev/
│   ├── main.tf
│   ├── terraform.tfvars    ← dev-specific values
│   └── backend.tf          ← dev state location
├── staging/
│   ├── main.tf
│   ├── terraform.tfvars
│   └── backend.tf
└── prod/
    ├── main.tf
    ├── terraform.tfvars
    └── backend.tf
```

**Why directories are better:** Separate state files, separate backends, can deploy independently, less risk of applying prod changes to dev.

---

## Terraform Import

Bring existing resources under Terraform management:

```bash
# 1. Write the resource block first
# main.tf:
# resource "aws_s3_bucket" "existing" {
#   bucket = "my-existing-bucket"
# }

# 2. Import it
terraform import aws_s3_bucket.existing my-existing-bucket

# 3. Run plan to see if config matches reality
terraform plan

# 4. Adjust your .tf file until plan shows no changes
```

---

## Terraform State Commands

```bash
# List all resources in state
terraform state list

# Show details of a resource
terraform state show aws_instance.web

# Move a resource (rename without destroying)
terraform state mv aws_instance.old aws_instance.new

# Remove from state (resource still exists in AWS, just unmanaged)
terraform state rm aws_instance.web

# Pull remote state to local file
terraform state pull > state.json
```

---

## Common Patterns

### Count vs for_each
```hcl
# count — by index (fragile if order changes)
resource "aws_subnet" "public" {
  count      = 3
  cidr_block = "10.0.${count.index}.0/24"
}

# for_each — by key (stable, preferred)
resource "aws_subnet" "public" {
  for_each   = toset(["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"])
  cidr_block = each.value
}
```

### Conditional Resources
```hcl
variable "create_bastion" {
  type    = bool
  default = false
}

resource "aws_instance" "bastion" {
  count         = var.create_bastion ? 1 : 0
  ami           = "ami-12345"
  instance_type = "t3.micro"
}
```

### Depends On
```hcl
resource "aws_iam_role_policy" "example" {
  # ...
}

resource "aws_instance" "web" {
  # Explicit dependency — wait for policy before creating instance
  depends_on = [aws_iam_role_policy.example]
}
```

---

## Best Practices

1. **Pin provider versions** — `version = "~> 5.0"` not `version = ">= 5.0"`
2. **Use remote state** — S3 + DynamoDB for teams
3. **Never edit state manually** — Use `terraform state` commands
4. **Use modules** — Don't repeat yourself
5. **Use variables** — No hardcoded values
6. **Use `terraform plan` before `apply`** — Always review
7. **Use `.gitignore`** — Ignore `.terraform/`, `*.tfstate`, `*.tfstate.backup`
8. **Tag everything** — Environment, team, project
9. **Small, focused configs** — Don't put everything in one state
10. **Lock versions** — Provider and module versions

### .gitignore for Terraform
```
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars       # If contains secrets
.terraform.lock.hcl  # Optional — some teams commit this
```

---

*Terraform is simple to start, deep to master. Practice with real infrastructure! 🏗️*
