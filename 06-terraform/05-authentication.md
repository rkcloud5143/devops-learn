# Terraform — Authentication & AWS Credentials 🔑

How Terraform connects to AWS. This is the first thing you need to understand.

---

## The Big Question: How Does Terraform Talk to AWS?

Terraform needs AWS credentials to create/modify resources. There are **4 ways** to provide them, from least secure to most secure.

---

## Method 1: Access Keys (Hardcoded) ❌ NEVER DO THIS

```hcl
provider "aws" {
  region     = "ca-central-1"
  access_key = "AKIAIOSFODNN7EXAMPLE"
  secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}
```

**Why it's bad:**
- Keys are in your code
- If you push to Git, anyone can use your AWS account
- AWS bots scan GitHub and will flag this immediately

**Pizza shop:** Writing the safe combination on the front door.

---

## Method 2: Environment Variables ✅ Good for CI/CD

```bash
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="ca-central-1"
```

Then your provider is clean:
```hcl
provider "aws" {
  region = "ca-central-1"
}
```

Terraform automatically reads these environment variables.

**When to use:** CI/CD pipelines (GitHub Actions, Jenkins) where secrets are injected.

**Pizza shop:** The manager whispers the safe code to the cook. Not written down, but still a code that could be overheard.

---

## Method 3: AWS Profile (Shared Credentials) ✅ Best for Local Development

### Step 1: Configure AWS CLI
```bash
aws configure --profile dev
# Enter: Access Key ID
# Enter: Secret Access Key
# Enter: Region (ca-central-1)
# Enter: Output format (json)
```

This creates `~/.aws/credentials`:
```ini
[dev]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[prod]
aws_access_key_id = AKIAI44QH8DHBEXAMPLE
aws_secret_access_key = je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
```

And `~/.aws/config`:
```ini
[profile dev]
region = ca-central-1

[profile prod]
region = us-east-1
```

### Step 2: Use Profile in Terraform
```hcl
provider "aws" {
  region  = "ca-central-1"
  profile = "dev"
}
```

Or set via environment variable:
```bash
export AWS_PROFILE=dev
terraform plan
```

### Multiple Accounts Example
```hcl
# Dev account
provider "aws" {
  region  = "ca-central-1"
  profile = "dev"
  alias   = "dev"
}

# Prod account
provider "aws" {
  region  = "us-east-1"
  profile = "prod"
  alias   = "prod"
}

# Use specific provider
resource "aws_s3_bucket" "dev_bucket" {
  provider = aws.dev
  bucket   = "my-dev-bucket"
}

resource "aws_s3_bucket" "prod_bucket" {
  provider = aws.prod
  bucket   = "my-prod-bucket"
}
```

**When to use:** Local development, multiple AWS accounts.

**Pizza shop:** Each manager has their own keycard. You pick which keycard to use.

---

## Method 4: IAM Role (AssumeRole) ✅✅ Best for Production & CI/CD

Instead of using long-lived access keys, assume a role with temporary credentials.

### How It Works
```
You (with basic credentials)
    │
    ▼
AssumeRole → AWS gives you temporary credentials (expire in 1 hour)
    │
    ▼
Terraform uses temporary credentials to create resources
```

### In Terraform
```hcl
provider "aws" {
  region = "ca-central-1"

  assume_role {
    role_arn     = "arn:aws:iam::123456789012:role/TerraformRole"
    session_name = "terraform-session"
  }
}
```

### Cross-Account with AssumeRole
```hcl
# Your credentials are in Account A
# You want to create resources in Account B

provider "aws" {
  region = "ca-central-1"

  assume_role {
    role_arn = "arn:aws:iam::987654321098:role/TerraformCrossAccountRole"
  }
}
```

### In CI/CD (GitHub Actions Example)
```yaml
# No access keys needed! Uses OIDC federation
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
    aws-region: ca-central-1
```

**When to use:** Production, CI/CD, cross-account access.

**Pizza shop:** Instead of giving the cook a permanent key, the manager gives them a temporary badge that expires at the end of their shift.

---

## Method 5: EC2 Instance Profile / ECS Task Role ✅✅ Best for AWS-to-AWS

If Terraform runs on an EC2 instance or in ECS, it automatically gets credentials from the instance profile or task role. No keys needed at all.

```hcl
# No credentials configured — Terraform uses instance role automatically
provider "aws" {
  region = "ca-central-1"
}
```

**When to use:** Terraform running inside AWS (Jenkins on EC2, CI/CD on ECS).

**Pizza shop:** The cook is already inside the kitchen — they don't need a key to get in.

---

## Summary: Which Method to Use?

```
┌─────────────────────────────────────────────────────────┐
│                  DECISION TREE                          │
│                                                         │
│  Running on your laptop?                                │
│    → Use AWS Profile (Method 3)                         │
│                                                         │
│  Running in CI/CD (GitHub Actions, Jenkins)?            │
│    → Use IAM Role with OIDC (Method 4)                  │
│    → Or Environment Variables (Method 2) as fallback    │
│                                                         │
│  Running on EC2/ECS inside AWS?                         │
│    → Use Instance Profile / Task Role (Method 5)        │
│                                                         │
│  Managing multiple AWS accounts?                        │
│    → Use AssumeRole (Method 4)                          │
│                                                         │
│  NEVER hardcode keys in .tf files (Method 1)            │
└─────────────────────────────────────────────────────────┘
```

| Method | Security | Use Case |
|--------|----------|----------|
| Hardcoded keys | ❌ Terrible | Never |
| Environment variables | ✅ OK | CI/CD pipelines |
| AWS Profile | ✅ Good | Local development |
| IAM Role (AssumeRole) | ✅✅ Best | Production, CI/CD, cross-account |
| Instance Profile | ✅✅ Best | Running inside AWS |

---

## Terragrunt Authentication

Terragrunt uses the **exact same methods** as Terraform. It's just a wrapper.

```hcl
# terragrunt.hcl
generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite"
  contents  = <<EOF
provider "aws" {
  region = "ca-central-1"

  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/TerraformRole"
  }
}
EOF
}
```

The difference: Terragrunt can **generate** the provider block for you, so you don't repeat it in every module.

---

*Start with profiles for learning, move to IAM roles for real projects! 🔐*
