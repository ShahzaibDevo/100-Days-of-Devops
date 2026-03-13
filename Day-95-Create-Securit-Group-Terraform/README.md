# 🚀 Day 95: Create Security Group Using Terraform

## 📋 Lab Overview
Use Terraform to create an AWS Security Group under the default VPC in `us-east-1` region.

---

## 🎯 Requirements
- Security Group name: `nautilus-sg`
- Description: `Security group for Nautilus App Servers`
- Inbound rule: HTTP — Port `80` — Source `0.0.0.0/0`
- Inbound rule: SSH — Port `22` — Source `0.0.0.0/0`
- Region: `us-east-1`
- Working directory: `/home/bob/terraform`

---

## 🛠️ Steps

### Step 1 — Open VS Code Terminal
Right-click under the **EXPLORER** section in VS Code → Select **Open in Integrated Terminal**

---

### Step 2 — Navigate to Working Directory
```bash
cd /home/bob/terraform
```

---

### Step 3 — Check Existing Files
```bash
ls -la
```
> ⚠️ Note: A `provider.tf` file already exists. Do NOT add `terraform {}` or `provider "aws"` blocks in `main.tf` to avoid duplicate config errors.

---

### Step 4 — Create `main.tf`
```bash
vi main.tf
```

Paste the following code:

```hcl
resource "aws_security_group" "nautilus_sg" {
  name        = "nautilus-sg"
  description = "Security group for Nautilus App Servers"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "nautilus-sg"
  }
}
```

Save and exit: `ESC` → `:wq` → `Enter`

---

### Step 5 — Initialize Terraform
```bash
terraform init
```

---

### Step 6 — Validate Configuration
```bash
terraform validate
```

---

### Step 7 — Plan the Deployment
```bash
terraform plan
```

---

### Step 8 — Apply the Configuration
```bash
terraform apply -auto-approve
```

---



## 💡 Key Learnings

- When a `provider.tf` already exists, **only put resources** in `main.tf`
- Duplicate `terraform {}` or `provider {}` blocks cause **init errors**
- Always check for stray characters (like backticks) at end of `.tf` files
- Terraform uses the **default VPC** automatically when no `vpc_id` is specified

---
