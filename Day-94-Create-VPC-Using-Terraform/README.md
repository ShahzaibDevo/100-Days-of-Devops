

# Day 94: Create VPC Using Terraform 

## What is a VPC?
A **Virtual Private Cloud (VPC)** is a logically isolated network in AWS where you can launch resources in a defined virtual network. It gives you full control over your network environment including IP address ranges, subnets, route tables, and gateways.

## What is Terraform?
**Terraform** is an Infrastructure as Code (IaC) tool by HashiCorp that allows you to define, provision, and manage cloud infrastructure using simple configuration files. Instead of manually clicking through AWS Console, you write code to create resources.

## Why use Terraform for VPC?
- **Reproducible** — same infrastructure every time
- **Version controlled** — track changes in Git
- **Automated** — no manual AWS Console clicks
- **Scalable** — easily extend to complex infrastructure

---

## Lab: Create VPC Using Terraform

### Prerequisites
- VS Code with Terraform working directory `/home/bob/terraform`
- `provider.tf` already configured with LocalStack endpoints

---

### Step 1: Open `main.tf` in VS Code
Navigate to `/home/bob/terraform` in the Explorer panel and open `main.tf`

### Step 2: Add VPC Resource
Press `Ctrl+A` to select all, delete existing content and paste:

```hcl
resource "aws_vpc" "datacenter_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "datacenter-vpc"
  }
}
```

### Step 3: Save the File
Press `Ctrl+S`

### Step 4: Open Integrated Terminal
Right-click under the EXPLORER section → **Open in Integrated Terminal**

### Step 5: Initialize Terraform
```bash
terraform init
```

### Step 6: Apply Configuration
```bash
terraform apply -auto-approve
```

### Step 7: Verify State
```bash
terraform state list
```

---

