# 🚀 Day 96 of #100DaysOfDevOps — Create EC2 Instance Using Terraform

## 📌 Overview

As part of the **Nautilus DevOps** infrastructure migration to AWS, today's challenge was to provision an EC2 instance using **Terraform** — an Infrastructure as Code (IaC) tool that lets us define cloud resources declaratively. The team is migrating in incremental steps, and this task was one of those smaller, manageable units.

---

## 🎯 Task Requirements

| Requirement         | Value                          |
|---------------------|-------------------------------|
| Instance Name (Tag) | `datacenter-ec2`              |
| AMI                 | `ami-0c101f26f147fa7fd` (Amazon Linux) |
| Instance Type       | `t2.micro`                    |
| Key Pair            | `datacenter-kp` (RSA)         |
| Security Group      | Default                        |
| Working Directory   | `/home/bob/terraform`          |
| Config File         | `main.tf` only                 |

---

## 🛠️ Solution — main.tf

```hcl
# Generate RSA private key
resource "tls_private_key" "datacenter_kp" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

# Create AWS Key Pair using the generated RSA public key
resource "aws_key_pair" "datacenter_kp" {
  key_name   = "datacenter-kp"
  public_key = tls_private_key.datacenter_kp.public_key_openssh
}

# Fetch the default VPC
data "aws_vpc" "default" {
  default = true
}

# Fetch the default security group
data "aws_security_group" "default" {
  name   = "default"
  vpc_id = data.aws_vpc.default.id
}

# Create EC2 Instance
resource "aws_instance" "datacenter_ec2" {
  ami                    = "ami-0c101f26f147fa7fd"
  instance_type          = "t2.micro"
  key_name               = aws_key_pair.datacenter_kp.key_name
  vpc_security_group_ids = [data.aws_security_group.default.id]

  tags = {
    Name = "datacenter-ec2"
  }
}
```

---

## ⚙️ Terraform Commands

```bash
# Step 1 — Initialize providers
terraform init

# Step 2 — Preview changes
terraform plan

# Step 3 — Apply and provision
terraform apply
```

---

## ✅ Result

```
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

| # | Resource Created                  |
|---|-----------------------------------|
| 1 | `tls_private_key.datacenter_kp`   |
| 2 | `aws_key_pair.datacenter_kp`      |
| 3 | `aws_instance.datacenter_ec2`     |

---

## 💡 Key Learnings

- **Terraform `data` sources** let you reference existing AWS resources (like the default VPC and security group) without hardcoding IDs — making configs more portable and reusable.
- **`tls_private_key` provider** generates an RSA key pair locally; the public key is then uploaded to AWS as a named key pair resource.
- **Duplicate provider blocks** across `.tf` files in the same directory will cause `terraform init` to fail — always keep provider configuration in one place (e.g., `provider.tf`).
- **`terraform plan`** is your best friend before applying — it shows exactly what will be created, changed, or destroyed with zero risk.
- Breaking large migrations into **small IaC tasks** makes infrastructure auditable, repeatable, and version-controlled.

---
