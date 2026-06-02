# Packer — Build Machine Images 📦

Create identical, pre-baked AMIs for EC2 instances.

---

## What is Packer?

```
Without Packer:
  1. Launch EC2 instance
  2. SSH in
  3. Install Docker, Nginx, monitoring agent...
  4. Configure everything
  5. Takes 10-15 minutes per server
  6. Every server is slightly different 😬

With Packer:
  1. Define everything in a template
  2. Packer builds an AMI with everything pre-installed
  3. Launch EC2 from AMI — ready in seconds
  4. Every server is IDENTICAL ✅

Terraform creates the infrastructure.
Packer creates the machine images that run ON that infrastructure.
```

---

## How It Works

```
┌─── Your Laptop ──────────────────────────────────────────┐
│                                                           │
│  packer build template.pkr.hcl                           │
│       │                                                   │
│       ▼                                                   │
│  1. Launches temporary EC2 instance                      │
│  2. SSHs into it                                         │
│  3. Runs provisioners (install software, copy files)     │
│  4. Creates AMI from the instance                        │
│  5. Terminates the temporary instance                    │
│  6. Outputs AMI ID                                       │
│                                                           │
│  Result: ami-0abc123def456 (ready to use in Terraform)   │
└───────────────────────────────────────────────────────────┘
```

---

## Packer Template (HCL)

```hcl
# amazon-linux.pkr.hcl

packer {
  required_plugins {
    amazon = {
      version = ">= 1.2.0"
      source  = "github.com/hashicorp/amazon"
    }
  }
}

# What to build FROM
source "amazon-ebs" "base" {
  ami_name      = "my-app-{{timestamp}}"
  instance_type = "t3.medium"
  region        = "ca-central-1"

  source_ami_filter {
    filters = {
      name                = "amzn2-ami-hvm-*-x86_64-gp2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    owners      = ["amazon"]
    most_recent = true
  }

  ssh_username = "ec2-user"

  tags = {
    Name        = "my-app"
    Environment = "production"
    ManagedBy   = "packer"
  }
}

# What to install
build {
  sources = ["source.amazon-ebs.base"]

  # Shell provisioner
  provisioner "shell" {
    inline = [
      "sudo yum update -y",
      "sudo yum install -y docker",
      "sudo systemctl enable docker",
      "sudo usermod -aG docker ec2-user",
      "sudo yum install -y amazon-cloudwatch-agent",
    ]
  }

  # Copy files
  provisioner "file" {
    source      = "configs/cloudwatch-config.json"
    destination = "/tmp/cloudwatch-config.json"
  }

  provisioner "shell" {
    inline = [
      "sudo mv /tmp/cloudwatch-config.json /opt/aws/amazon-cloudwatch-agent/etc/",
    ]
  }

  # Ansible provisioner (alternative)
  # provisioner "ansible" {
  #   playbook_file = "ansible/playbook.yml"
  # }
}
```

---

## Commands

```bash
packer init .                          # Download plugins
packer fmt .                           # Format template
packer validate .                      # Validate template
packer build .                         # Build the AMI
packer build -var 'env=prod' .         # With variables
packer build -only='amazon-ebs.base' . # Specific source only
```

---

## Packer + Terraform Workflow

```
1. Packer builds AMI → ami-0abc123
2. Terraform uses that AMI for EC2/ASG

# In Terraform:
data "aws_ami" "my_app" {
  most_recent = true
  owners      = ["self"]
  filter {
    name   = "name"
    values = ["my-app-*"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.my_app.id
  instance_type = "t3.medium"
}
```

---

## When to Use Packer

```
✅ Use Packer:
  - Immutable infrastructure (never modify running servers)
  - Consistent AMIs across environments
  - Pre-bake dependencies (faster boot time)
  - Golden images for Auto Scaling Groups

❌ Don't need Packer:
  - Containers (use Docker instead)
  - Simple EC2 with user_data
  - Serverless (Lambda)
```

---

*Packer = bake once, launch many. Pairs perfectly with Terraform! 📦*
