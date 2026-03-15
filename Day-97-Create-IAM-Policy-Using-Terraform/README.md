# Day 97: Create IAM Policy Using Terraform



## Task Overview

Create an IAM policy named `iampolicy_james` in `us-east-1` region using Terraform.
The policy must allow read-only access to the EC2 console — view instances, AMIs, and snapshots.

- **Working Directory:** `/home/bob/terraform`
- **Config File:** `main.tf` (only)

---

## Project Structure

```
/home/bob/terraform/
├── provider.tf       # AWS provider config (pre-existing)
└── main.tf           # IAM policy resources (created by us)
```

---

## provider.tf (pre-existing)

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

---

## main.tf (solution)

```hcl
data "aws_iam_policy_document" "ec2_read_only" {
  statement {
    sid    = "EC2ReadOnlyAccess"
    effect = "Allow"

    actions = [
      "ec2:Describe*",
      "ec2:GetConsoleOutput",
      "ec2:GetConsoleScreenshot"
    ]

    resources = ["*"]
  }
}

resource "aws_iam_policy" "iampolicy_james" {
  name        = "iampolicy_james"
  description = "Read-only access to EC2 console - view instances, AMIs, and snapshots"
  policy      = data.aws_iam_policy_document.ec2_read_only.json
}
```

---

## Steps to Execute

### Step 1 — Navigate to working directory
```bash
cd /home/bob/terraform
```

### Step 2 — Check existing files
```bash
ls *.tf
# provider.tf already exists — do NOT duplicate provider config in main.tf
```


### Step 4 — Initialize Terraform
```bash
terraform init
```

### Step 5 — Preview the plan
```bash
terraform plan
```

### Step 6 — Apply the configuration
```bash
terraform apply -auto-approve
```

---

##

## Key Concepts

| Concept | Description |
|---|---|
| `aws_iam_policy_document` | Terraform data source that generates a JSON IAM policy from HCL |
| `ec2:Describe*` | Wildcard covering all read-only EC2 describe actions |
| `ec2:GetConsoleOutput` | Allows viewing EC2 instance console output |
| `ec2:GetConsoleScreenshot` | Allows viewing EC2 instance console screenshot |
| `resources = ["*"]` | Required — EC2 Describe actions do not support resource-level restrictions |
| No duplicate providers | Terraform merges all `.tf` files — only one provider block allowed per module |

---
