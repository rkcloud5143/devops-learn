# AWS IAM — Deep Dive

---

## THE BIG PICTURE

```
┌─── AWS Account ──────────────────────────────────────────────────┐
│                                                                   │
│  Root User (email + password)                                    │
│  └── Has FULL access to everything — NEVER use for daily work    │
│  └── Enable MFA immediately                                     │
│  └── Only use for: billing, account settings, first IAM admin    │
│                                                                   │
│  ┌─── IAM (Identity & Access Management) ─────────────────────┐ │
│  │                                                              │ │
│  │  WHO can access?          WHAT can they do?                 │ │
│  │  ┌──────────────┐        ┌──────────────┐                  │ │
│  │  │  Identities  │        │   Policies   │                  │ │
│  │  │  - Users     │───────►│  (JSON docs) │                  │ │
│  │  │  - Groups    │        │              │                  │ │
│  │  │  - Roles     │        │  Allow/Deny  │                  │ │
│  │  └──────────────┘        └──────────────┘                  │ │
│  │                                                              │ │
│  └──────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────┘
```

---

## 1. IAM USERS — Types of Access

```
IAM User = a person OR a service that needs AWS access

┌─── IAM User ──────────────────────────────────────────────────┐
│                                                                │
│  Access Type 1: Console Access (for humans)                   │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  - Username + Password                                  │   │
│  │  - Used to: login to AWS web console (browser)          │   │
│  │  - Enable MFA (Multi-Factor Authentication)             │   │
│  │  - Example: developer logs into console to check EC2    │   │
│  │                                                         │   │
│  │  Console URL: https://123456789012.signin.aws.amazon.com│   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
│  Access Type 2: Programmatic Access (CLI / SDK / API)         │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  - Access Key ID + Secret Access Key                    │   │
│  │  - Used to: AWS CLI, Terraform, scripts, applications   │   │
│  │  - NEVER share or commit to Git                         │   │
│  │  - Rotate regularly (every 90 days recommended)         │   │
│  │                                                         │   │
│  │  Example:                                               │   │
│  │  Access Key ID:     AKIAIOSFODNN7EXAMPLE                │   │
│  │  Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLE│  │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
│  Access Type 3: Both (Console + Programmatic)                 │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  - User gets password AND access keys                   │   │
│  │  - Common for DevOps engineers who use both              │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
└────────────────────────────────────────────────────────────────┘

Best Practices:
- One IAM user per person (never share accounts)
- Use groups to assign permissions (not directly to users)
- Enable MFA on ALL users
- Use access keys only when needed, prefer roles instead
- Delete unused users and keys
```

### When to Use IAM Users vs Roles
```
┌──────────────────────────────┬──────────────────────────────────┐
│ Use IAM USER when:           │ Use IAM ROLE when:               │
├──────────────────────────────┼──────────────────────────────────┤
│ A person needs console login │ An AWS service needs permissions │
│ A CI/CD tool needs keys      │ EC2 needs to access S3           │
│ (but prefer roles if possible│ Lambda needs to write to DynamoDB│
│                              │ Cross-account access             │
│                              │ Temporary access needed          │
└──────────────────────────────┴──────────────────────────────────┘
```

---

## 2. IAM GROUPS

```
┌─── IAM Groups ────────────────────────────────────────────────┐
│                                                                │
│  Group = collection of users                                  │
│  Attach policies to groups, NOT to individual users           │
│                                                                │
│  ┌─── Developers Group ──────────────────────────────────┐   │
│  │  Members: dev-1, dev-2, dev-3                          │   │
│  │  Policies: EC2ReadOnly, S3ReadWrite, CloudWatchRead    │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
│  ┌─── DevOps Group ──────────────────────────────────────┐   │
│  │  Members: devops-1, devops-2                           │   │
│  │  Policies: EC2FullAccess, S3FullAccess, IAMReadOnly   │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
│  ┌─── Admins Group ──────────────────────────────────────┐   │
│  │  Members: admin-1                                      │   │
│  │  Policies: AdministratorAccess                         │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
│  Rules:                                                       │
│  - A user can belong to multiple groups                       │
│  - Groups CANNOT be nested (no group inside a group)          │
│  - Groups are for users only (not for roles or services)      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 3. IAM ROLES — Deep Dive

```
Role = temporary permissions that can be "assumed" by:
       - AWS services (EC2, Lambda, ECS)
       - Other AWS accounts (cross-account)
       - Federated users (SSO, Google, Active Directory)

Unlike users, roles have NO password or access keys.
They provide temporary security credentials (auto-rotated).
```

### Types of Roles

```
┌─── 1. AWS Service Role ──────────────────────────────────────┐
│                                                               │
│  WHO assumes it: AWS services (EC2, Lambda, ECS, etc.)       │
│                                                               │
│  Example: EC2 instance needs to read from S3                 │
│                                                               │
│  ┌──────────┐    assumes     ┌──────────────────┐            │
│  │   EC2    │ ─────────────► │ EC2-S3-ReadRole   │            │
│  │ Instance │                │                    │            │
│  └──────────┘                │ Trust: ec2.amazonaws.com       │
│                              │ Permissions: s3:GetObject      │
│                              └──────────────────┘            │
│                                                               │
│  This is the MOST COMMON role type in DevOps                 │
│  Used for: EC2, Lambda, ECS tasks, CodeBuild, etc.           │
│                                                               │
│  Trust Policy (who can assume this role):                    │
│  {                                                            │
│    "Effect": "Allow",                                        │
│    "Principal": {                                            │
│      "Service": "ec2.amazonaws.com"                          │
│    },                                                        │
│    "Action": "sts:AssumeRole"                                │
│  }                                                            │
└───────────────────────────────────────────────────────────────┘

┌─── 2. Cross-Account Role ────────────────────────────────────┐
│                                                               │
│  WHO assumes it: Users/roles from ANOTHER AWS account        │
│                                                               │
│  Example: Dev account needs to deploy to Prod account        │
│                                                               │
│  Account A (Dev)          Account B (Prod)                   │
│  ┌──────────┐   assumes   ┌──────────────────┐              │
│  │ Dev User │ ──────────► │ ProdDeployRole    │              │
│  └──────────┘             │                    │              │
│                           │ Trust: Account A   │              │
│                           │ Permissions: ECS   │              │
│                           └──────────────────┘              │
│                                                               │
│  Trust Policy:                                               │
│  {                                                            │
│    "Effect": "Allow",                                        │
│    "Principal": {                                            │
│      "AWS": "arn:aws:iam::111111111111:root"                 │
│    },                                                        │
│    "Action": "sts:AssumeRole"                                │
│  }                                                            │
└───────────────────────────────────────────────────────────────┘

┌─── 3. Identity Provider Role (Federation) ───────────────────┐
│                                                               │
│  WHO assumes it: External users (SSO, Google, SAML, OIDC)   │
│                                                               │
│  Example: Company uses Okta/Azure AD for SSO                │
│                                                               │
│  ┌──────────┐   SAML/OIDC   ┌──────────────────┐            │
│  │  Okta /  │ ─────────────► │ FederatedRole     │            │
│  │ Azure AD │                │                    │            │
│  └──────────┘                │ Trust: Okta IDP    │            │
│                              │ Permissions: varies│            │
│                              └──────────────────┘            │
│                                                               │
│  Types:                                                      │
│  - SAML 2.0: corporate SSO (Okta, Azure AD, OneLogin)       │
│  - Web Identity / OIDC: Google, Facebook, GitHub Actions     │
│  - AWS SSO / IAM Identity Center: AWS's own SSO solution     │
│                                                               │
│  GitHub Actions uses OIDC to assume AWS roles (no keys!)     │
└───────────────────────────────────────────────────────────────┘

┌─── 4. Service-Linked Role ───────────────────────────────────┐
│                                                               │
│  WHO creates it: AWS automatically                           │
│  WHO assumes it: Specific AWS service                        │
│                                                               │
│  - Pre-defined by AWS, you can't modify the permissions      │
│  - Created automatically when you enable certain services    │
│  - Example: AWSServiceRoleForAutoScaling                     │
│  - Example: AWSServiceRoleForElasticLoadBalancing            │
│                                                               │
│  You'll see these in IAM → Roles, don't delete them!        │
└───────────────────────────────────────────────────────────────┘
```

### How Role Assumption Works (STS)
```
┌──────────┐                    ┌─────────┐
│  EC2     │  1. AssumeRole     │  AWS    │
│ Instance │ ──────────────────►│  STS    │  (Security Token Service)
│          │                    │         │
│          │  2. Returns:       │         │
│          │ ◄──────────────────│         │
│          │  - Access Key      │         │
│          │  - Secret Key      │         │
│          │  - Session Token   │         │
│          │  - Expiration      │         │
└──────────┘  (temporary!)      └─────────┘

These credentials auto-rotate. No keys to manage!
This is why roles are MORE SECURE than access keys.
```

---

## 4. IAM POLICIES — Deep Dive

### Policy Structure (JSON)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Read",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}

Breaking it down:
- Version:   always "2012-10-17" (don't change this)
- Sid:       optional description/label
- Effect:    "Allow" or "Deny"
- Action:    what API calls are permitted (s3:GetObject, ec2:*)
- Resource:  which specific resources (ARN)
- Condition: optional extra restrictions (IP, time, MFA, tags)
```

### Three Types of Policies

```
┌─── 1. AWS Managed Policies ──────────────────────────────────┐
│                                                               │
│  Created and maintained by AWS                               │
│  You CANNOT edit them                                        │
│  Reusable across multiple users/groups/roles                 │
│  AWS updates them when new services launch                   │
│                                                               │
│  Common ones to know:                                        │
│  ┌────────────────────────────────┬──────────────────────┐   │
│  │ Policy Name                    │ What it does         │   │
│  ├────────────────────────────────┼──────────────────────┤   │
│  │ AdministratorAccess            │ Full access to ALL   │   │
│  │ PowerUserAccess                │ Full except IAM      │   │
│  │ ReadOnlyAccess                 │ Read-only everything │   │
│  │ AmazonEC2FullAccess            │ Full EC2 access      │   │
│  │ AmazonEC2ReadOnlyAccess        │ Read-only EC2        │   │
│  │ AmazonS3FullAccess             │ Full S3 access       │   │
│  │ AmazonS3ReadOnlyAccess         │ Read-only S3         │   │
│  │ AmazonVPCFullAccess            │ Full VPC access      │   │
│  │ AmazonRDSFullAccess            │ Full RDS access      │   │
│  │ CloudWatchFullAccess           │ Full CloudWatch      │   │
│  │ AmazonECS_FullAccess           │ Full ECS access      │   │
│  │ AWSLambda_FullAccess           │ Full Lambda access   │   │
│  └────────────────────────────────┴──────────────────────┘   │
│                                                               │
│  Icon in console: orange AWS cube icon                       │
│  Good for: quick setup, common use cases                     │
│  Risk: often too broad (violates least privilege)            │
└───────────────────────────────────────────────────────────────┘

┌─── 2. Customer Managed Policies ─────────────────────────────┐
│                                                               │
│  Created by YOU (your organization)                          │
│  You CAN edit, version, and update them                      │
│  Reusable across multiple users/groups/roles                 │
│  Up to 5 versions (for rollback)                             │
│                                                               │
│  Example: Custom policy for your app team                    │
│  {                                                            │
│    "Version": "2012-10-17",                                  │
│    "Statement": [                                            │
│      {                                                        │
│        "Effect": "Allow",                                    │
│        "Action": [                                           │
│          "s3:GetObject",                                     │
│          "s3:PutObject"                                      │
│        ],                                                    │
│        "Resource": "arn:aws:s3:::my-app-bucket/*"            │
│      },                                                      │
│      {                                                        │
│        "Effect": "Allow",                                    │
│        "Action": [                                           │
│          "ec2:DescribeInstances",                            │
│          "ec2:StartInstances",                               │
│          "ec2:StopInstances"                                 │
│        ],                                                    │
│        "Resource": "*",                                      │
│        "Condition": {                                        │
│          "StringEquals": {                                   │
│            "ec2:ResourceTag/Environment": "dev"              │
│          }                                                    │
│        }                                                      │
│      }                                                        │
│    ]                                                          │
│  }                                                            │
│                                                               │
│  Good for: least privilege, specific to your needs           │
│  THIS IS WHAT YOU SHOULD USE IN PRODUCTION                   │
└───────────────────────────────────────────────────────────────┘

┌─── 3. Inline Policies ───────────────────────────────────────┐
│                                                               │
│  Embedded directly into a single user, group, or role        │
│  NOT reusable — dies when the user/role is deleted           │
│  Cannot be versioned                                         │
│                                                               │
│  ┌──────────┐                                                │
│  │ User: A  │ ◄── inline policy embedded here                │
│  │          │     (only applies to User A)                   │
│  └──────────┘                                                │
│                                                               │
│  When to use:                                                │
│  - Strict 1:1 relationship (policy MUST go with this entity) │
│  - Emergency/temporary access                                │
│  - You want to ensure policy is deleted with the user/role   │
│                                                               │
│  When NOT to use:                                            │
│  - Most of the time! Prefer managed policies                 │
│  - Hard to audit and manage at scale                         │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### Policy Comparison Table
```
┌─────────────────────┬──────────────┬──────────────┬──────────────┐
│                     │ AWS Managed  │ Customer     │ Inline       │
│                     │              │ Managed      │              │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Created by          │ AWS          │ You          │ You          │
│ Editable            │ No           │ Yes          │ Yes          │
│ Reusable            │ Yes          │ Yes          │ No           │
│ Versioning          │ AWS manages  │ Up to 5      │ No           │
│ Deleted with entity │ No           │ No           │ Yes          │
│ Best for            │ Common cases │ Production   │ Exceptions   │
│ Least privilege     │ Often too    │ Yes (custom) │ Yes          │
│                     │ broad        │              │              │
└─────────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 5. POLICY EVALUATION LOGIC

```
Request comes in
      │
      ▼
┌─────────────────┐     Yes
│ Explicit DENY?  │ ──────────► ACCESS DENIED
└────────┬────────┘
         │ No
         ▼
┌─────────────────┐     Yes
│ Explicit ALLOW? │ ──────────► ACCESS ALLOWED
└────────┬────────┘
         │ No
         ▼
    ACCESS DENIED
   (implicit deny)

KEY RULE: Deny ALWAYS wins over Allow
If any policy says Deny, it doesn't matter how many say Allow.
```

### Example: Multiple Policies on a User
```
User "dev-1" is in group "Developers"

Group policy says:     "Allow s3:*"        (all S3 actions)
Inline policy says:    "Deny s3:DeleteBucket"

Result:
- s3:GetObject     → ALLOWED (group policy)
- s3:PutObject     → ALLOWED (group policy)
- s3:DeleteBucket  → DENIED  (explicit deny wins!)
```

---

## 6. RESOURCE-BASED POLICIES

```
Identity-based policy:  attached to user/group/role
                        "This user CAN access S3"

Resource-based policy:  attached to the resource itself
                        "This S3 bucket ALLOWS this user"

┌─── S3 Bucket Policy (resource-based) ────────────────────────┐
│                                                               │
│  {                                                            │
│    "Version": "2012-10-17",                                  │
│    "Statement": [                                            │
│      {                                                        │
│        "Effect": "Allow",                                    │
│        "Principal": {                                        │
│          "AWS": "arn:aws:iam::111111111111:role/MyRole"       │
│        },                                                    │
│        "Action": "s3:GetObject",                             │
│        "Resource": "arn:aws:s3:::my-bucket/*"                │
│      }                                                        │
│    ]                                                          │
│  }                                                            │
│                                                               │
│  Services that support resource-based policies:              │
│  - S3 (bucket policies)                                      │
│  - SQS (queue policies)                                      │
│  - SNS (topic policies)                                      │
│  - Lambda (function policies)                                │
│  - KMS (key policies)                                        │
│  - ECR (repository policies)                                 │
└───────────────────────────────────────────────────────────────┘
```

---

## 7. PERMISSION BOUNDARIES

```
Permission Boundary = maximum permissions a user/role CAN have

Think of it as a "ceiling" on permissions:

┌─── Without Boundary ─────────┐  ┌─── With Boundary ─────────────┐
│                               │  │                                │
│  Policy: AdministratorAccess  │  │  Policy: AdministratorAccess   │
│  Effective: FULL access       │  │  Boundary: S3 + EC2 only       │
│                               │  │  Effective: S3 + EC2 only      │
│                               │  │                                │
│  (no limit)                   │  │  (boundary limits the policy)  │
└───────────────────────────────┘  └────────────────────────────────┘

Effective permissions = Policy ∩ Boundary (intersection)

Use case: Let junior admins create IAM roles but limit
what permissions those roles can have.
```

---

## 8. IAM BEST PRACTICES CHECKLIST

```
Security:
├── [ ] Enable MFA on root account
├── [ ] Enable MFA on all IAM users
├── [ ] Never use root account for daily tasks
├── [ ] Use strong password policy
└── [ ] Enable CloudTrail to audit all IAM actions

Access Management:
├── [ ] Follow least privilege (only give what's needed)
├── [ ] Use groups to assign permissions
├── [ ] Use roles for AWS services (not access keys)
├── [ ] Use customer managed policies (not AWS managed) in prod
├── [ ] Review and remove unused users/roles/keys regularly
└── [ ] Use IAM Access Analyzer to find unused permissions

Keys & Credentials:
├── [ ] Rotate access keys every 90 days
├── [ ] Never hardcode keys in code
├── [ ] Use environment variables or IAM roles instead
├── [ ] Delete access keys for console-only users
└── [ ] Use AWS Secrets Manager for application secrets

Cross-Account:
├── [ ] Use roles for cross-account access (not sharing keys)
└── [ ] Use AWS Organizations for multi-account management
```

---

## 9. COMMON INTERVIEW SCENARIOS

**Q: An EC2 instance needs to read from S3. How do you set it up?**
```
Answer: Create an IAM Role (not a user!)
1. Create IAM role with trust policy for ec2.amazonaws.com
2. Attach a policy with s3:GetObject permission
3. Attach the role to the EC2 instance (Instance Profile)
4. Application on EC2 uses AWS SDK — credentials are automatic

NEVER put access keys on an EC2 instance!
```

**Q: A developer needs read-only access to production S3 but full access to dev S3.**
```
Answer: Customer managed policy with conditions
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::dev-*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": "arn:aws:s3:::prod-*"
    }
  ]
}
```

**Q: What's the difference between IAM role and IAM user?**
```
User:
- Long-term credentials (password, access keys)
- For people or long-running services
- Credentials don't expire automatically

Role:
- Temporary credentials (auto-rotated by STS)
- For AWS services, cross-account, federation
- Credentials expire (15 min to 12 hours)
- More secure — no keys to leak
```
