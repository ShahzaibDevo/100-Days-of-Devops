# 🚀 Day 99/100 - Attach IAM Policy for DynamoDB Access Using Terraform

## 📋 Overview
The DevOps team was tasked with creating a secure DynamoDB table and enforcing fine-grained access control using IAM — allowing restricted read-only access from trusted AWS services using **Terraform**.

---

## 🗂️ File Structure
```
/home/bob/terraform/
├── provider.tf       ← terraform block + AWS provider (LocalStack)
├── main.tf           ← all resources: table, role, policy, attachment
├── variables.tf      ← variable declarations
├── terraform.tfvars  ← actual values for variables
└── outputs.tf        ← output definitions
```

---

## 📄 provider.tf
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.91.0"
    }
  }
}

provider "aws" {
  region                      = "us-east-1"
  skip_credentials_validation = true
  skip_requesting_account_id  = true
  s3_use_path_style           = true

  endpoints {
    ec2            = "http://aws:4566"
    apigateway     = "http://aws:4566"
    cloudformation = "http://aws:4566"
    cloudwatch     = "http://aws:4566"
    dynamodb       = "http://aws:4566"
    es             = "http://aws:4566"
    firehose       = "http://aws:4566"
    iam            = "http://aws:4566"
    kinesis        = "http://aws:4566"
    lambda         = "http://aws:4566"
    route53        = "http://aws:4566"
    redshift       = "http://aws:4566"
    s3             = "http://aws:4566"
    secretsmanager = "http://aws:4566"
    ses            = "http://aws:4566"
    sns            = "http://aws:4566"
    sqs            = "http://aws:4566"
    ssm            = "http://aws:4566"
    stepfunctions  = "http://aws:4566"
    sts            = "http://aws:4566"
    rds            = "http://aws:4566"
  }
}
```

> ⚠️ **Important:** Keep `terraform {}` and `provider {}` blocks in ONE file only to avoid duplicate provider errors.

---

## 📄 variables.tf
```hcl
variable "KKE_TABLE_NAME" {
  description = "Name of the DynamoDB table"
  type        = string
}

variable "KKE_ROLE_NAME" {
  description = "Name of the IAM role"
  type        = string
}

variable "KKE_POLICY_NAME" {
  description = "Name of the IAM policy"
  type        = string
}
```

---

## 📄 terraform.tfvars
```hcl
KKE_TABLE_NAME  = "xfusion-table"
KKE_ROLE_NAME   = "xfusion-role"
KKE_POLICY_NAME = "xfusion-readonly-policy"
```

---

## 📄 main.tf
```hcl
# 1. DynamoDB Table
resource "aws_dynamodb_table" "xfusion_table" {
  name         = var.KKE_TABLE_NAME
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

# 2. IAM Role
resource "aws_iam_role" "xfusion_role" {
  name = var.KKE_ROLE_NAME

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = { Service = "lambda.amazonaws.com" }
        Action    = "sts:AssumeRole"
      }
    ]
  })
}

# 3. IAM Policy – read-only, scoped to specific table ARN
resource "aws_iam_policy" "xfusion_readonly_policy" {
  name        = var.KKE_POLICY_NAME
  description = "Read-only access to the xfusion DynamoDB table"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:Scan",
          "dynamodb:Query"
        ]
        Resource = aws_dynamodb_table.xfusion_table.arn
      }
    ]
  })
}

# 4. Attach policy to role
resource "aws_iam_role_policy_attachment" "xfusion_policy_attach" {
  role       = aws_iam_role.xfusion_role.name
  policy_arn = aws_iam_policy.xfusion_readonly_policy.arn
}
```

---

## 📄 outputs.tf
```hcl
output "kke_dynamodb_table" {
  description = "Name of the DynamoDB table"
  value       = aws_dynamodb_table.xfusion_table.name
}

output "kke_iam_role_name" {
  description = "Name of the IAM role"
  value       = aws_iam_role.xfusion_role.name
}

output "kke_iam_policy_name" {
  description = "Name of the IAM policy"
  value       = aws_iam_policy.xfusion_readonly_policy.name
}
```

---

## ▶️ Execution
```bash
# Step 1 - Initialize
terraform init

# Step 2 - Validate
terraform validate

# Step 3 - Apply
terraform apply -auto-approve

# Step 4 - Verify (must show No changes)
terraform plan
```

### ✅ Apply Output
```
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:
kke_dynamodb_table  = "xfusion-table"
kke_iam_policy_name = "xfusion-readonly-policy"
kke_iam_role_name   = "xfusion-role"
```

### ✅ Plan Output
```
No changes. Your infrastructure matches the configuration.
```

---


---

## 💡 Key Learnings

| Concept | Detail |
|---|---|
| 🔒 Least Privilege | Policy grants only `GetItem`, `Scan`, `Query` — no write/delete access |
| 🎯 Resource Scoping | `Resource` is set to exact table ARN, not `*` wildcard |
| 📦 File Separation | Split provider, variables, outputs, and resources into separate `.tf` files |
| 🏠 LocalStack | Test AWS infrastructure locally without real AWS costs or credentials |
| 🐛 Duplicate Provider | Only ONE `terraform {}` and `provider {}` block allowed per Terraform module |

---
