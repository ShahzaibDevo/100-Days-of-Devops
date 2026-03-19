

# Day 100: Create and Configure Alarm Using CloudWatch Using Terraform 🚀

## 📋 Lab Overview
In this lab, we set up an EC2 instance and configured a CloudWatch alarm to monitor CPU utilization using Terraform. The alarm triggers when CPU usage exceeds 90% for one consecutive 5-minute period and sends notifications via SNS.

---

## 🎯 Objectives
- Launch an EC2 instance using Terraform
- Create a CloudWatch alarm for CPU monitoring
- Integrate SNS topic for alarm notifications
- Output key resource values using `outputs.tf`

---

## 🏗️ Architecture

```
┌─────────────────┐     CPU > 90%      ┌──────────────────────┐
│   EC2 Instance  │ ──────────────────▶ │  CloudWatch Alarm    │
│  (xfusion-ec2)  │                    │  (xfusion-alarm)     │
└─────────────────┘                    └──────────┬───────────┘
                                                  │
                                                  ▼
                                       ┌──────────────────────┐
                                       │     SNS Topic        │
                                       │ (xfusion-sns-topic)  │
                                       └──────────────────────┘
```


---

## 📝 Configuration Files

### `main.tf`
```hcl
resource "aws_sns_topic" "sns_topic" {
  name = "xfusion-sns-topic"
}

resource "aws_instance" "xfusion_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"

  tags = {
    Name = "xfusion-ec2"
  }
}

resource "aws_cloudwatch_metric_alarm" "xfusion_alarm" {
  alarm_name          = "xfusion-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300
  statistic           = "Average"
  threshold           = 90

  dimensions = {
    InstanceId = aws_instance.xfusion_ec2.id
  }

  alarm_actions     = [aws_sns_topic.sns_topic.arn]
  alarm_description = "Alarm when CPU exceeds 90% for 5 minutes"
}
```

### `outputs.tf`
```hcl
output "KKE_instance_name" {
  value = aws_instance.xfusion_ec2.tags["Name"]
}

output "KKE_alarm_name" {
  value = aws_cloudwatch_metric_alarm.xfusion_alarm.alarm_name
}
```

---

## 🚀 Deployment Steps

### 1. Navigate to Terraform Directory
```bash
cd /home/bob/terraform
```

### 2. Check Existing Files
```bash
ls -la
cat provider.tf
```

### 3. Initialize Terraform
```bash
terraform init
```

### 4. Plan Infrastructure
```bash
terraform plan
```

### 5. Apply Configuration
```bash
terraform apply -auto-approve
```



## ⚙️ CloudWatch Alarm Specifications

| Parameter | Value |
|---|---|
| Alarm Name | `xfusion-alarm` |
| Metric | `CPUUtilization` |
| Namespace | `AWS/EC2` |
| Statistic | `Average` |
| Threshold | `>= 90%` |
| Period | `300 seconds (5 min)` |
| Evaluation Periods | `1` |
| Alarm Action | `xfusion-sns-topic` |

---


---

## 📚 Key Learnings

- Always check for existing `.tf` files before adding provider blocks to avoid **duplicate provider** errors
- If a resource is managed by Terraform state, removing it from config will **destroy** it — always keep managed resources in config
- Use `data` source only for resources **not** managed by Terraform
- `terraform state list` helps identify what's already tracked in state
- The `outputs.tf` file must exist as a **separate physical file** for lab validators to pass

---