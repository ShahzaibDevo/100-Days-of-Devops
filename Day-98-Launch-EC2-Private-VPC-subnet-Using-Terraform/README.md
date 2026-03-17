#  Day 98: Launch EC2 in Private VPC Subnet Using Terraform




## 🎯 What I Built Today

Provisioned a **fully private AWS VPC** with a private subnet and launched an EC2 instance inside it — all using **Terraform Infrastructure as Code**. The instance is isolated from the internet and accessible only from within the VPC CIDR block.

---

## 🏗️ Architecture Overview

```
AWS Cloud
└── VPC: xfusion-priv-vpc (10.0.0.0/16)
    └── Subnet: xfusion-priv-subnet (10.0.1.0/24)
        ├── Security Group: xfusion-priv-sg
        │   └── Ingress: Allow all from 10.0.0.0/16 only
        └── EC2: xfusion-priv-ec2 (t2.micro)
            └── No public IP assigned
```

---

## 📁 File Structure

```
/home/bob/terraform/
├── main.tf        # VPC, Subnet, Security Group, EC2
├── variables.tf   # Input variables
└── outputs.tf     # Output values
```

---

## 📄 File 1: variables.tf

```hcl
variable "KKE_VPC_CIDR" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "KKE_SUBNET_CIDR" {
  description = "CIDR block for the subnet"
  type        = string
  default     = "10.0.1.0/24"
}
```

---

## 📄 File 2: main.tf

```hcl
resource "aws_vpc" "xfusion_priv_vpc" {
  cidr_block           = var.KKE_VPC_CIDR
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "xfusion-priv-vpc"
  }
}

resource "aws_subnet" "xfusion_priv_subnet" {
  vpc_id                  = aws_vpc.xfusion_priv_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "xfusion-priv-subnet"
  }
}

resource "aws_security_group" "xfusion_priv_sg" {
  name        = "xfusion-priv-sg"
  description = "Allow access only from within the VPC CIDR"
  vpc_id      = aws_vpc.xfusion_priv_vpc.id

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "xfusion-priv-sg"
  }
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "xfusion_priv_ec2" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.xfusion_priv_subnet.id
  vpc_security_group_ids      = [aws_security_group.xfusion_priv_sg.id]
  associate_public_ip_address = false

  tags = {
    Name = "xfusion-priv-ec2"
  }

  lifecycle {
    ignore_changes = [vpc_security_group_ids]
  }
}
```

---

## 📄 File 3: outputs.tf

```hcl
output "KKE_vpc_name" {
  description = "Name of the VPC"
  value       = aws_vpc.xfusion_priv_vpc.tags["Name"]
}

output "KKE_subnet_name" {
  description = "Name of the subnet"
  value       = aws_subnet.xfusion_priv_subnet.tags["Name"]
}

output "KKE_ec2_private" {
  description = "Name of the EC2 instance"
  value       = aws_instance.xfusion_priv_ec2.tags["Name"]
}
```

---

## ▶️ Commands to Deploy

```bash
# Step 1 — Initialize Terraform
terraform init

# Step 2 — Preview changes
terraform plan

# Step 3 — Apply configuration
terraform apply -auto-approve

# Step 4 — Verify no drift
terraform plan
```

---

## ✅ Final Output

```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:
KKE_ec2_private = "xfusion-priv-ec2"
KKE_subnet_name = "xfusion-priv-subnet"
KKE_vpc_name    = "xfusion-priv-vpc"
```

```
No changes. Your infrastructure matches the configuration.
```

---

## 💡 Key Concepts Learned

- **Private VPC** — Resources isolated from the public internet
- **map_public_ip_on_launch = false** — Disables auto-assign public IP on subnet
- **Security Group ingress** — Restricted to VPC CIDR `10.0.0.0/16` only
- **data source aws_ami** — Dynamically fetches the latest Amazon Linux 2 AMI
- **lifecycle ignore_changes** — Prevents false drift on security group attachments
- **Terraform outputs** — Expose resource names after apply

---
