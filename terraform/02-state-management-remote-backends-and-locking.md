# Terraform State Management, Remote Backends & State Locking

---

## 1. Role of the State File (`terraform.tfstate`)

The state file is the single source of truth mapping your HCL declarations to real-world cloud resource IDs:
* **Resource Metadata:** Maps logical resource addresses (`aws_instance.web`) to cloud provider resource IDs (`i-0a1b2c3d4e5f6g7h8`).
* **Performance Cache:** Speeds up planning by caching resource attributes between runs.
* **Dependency Graph:** Tracks dependencies to determine parallelization and safe teardown ordering.

---

## 2. Remote State Backend Architecture (AWS S3 + DynamoDB)

```text
[Developer / CI/CD] -> [terraform apply]
                            |
             1. Acquire Lock via DynamoDB (LockID)
                            |
             2. Fetch latest State from S3 (Encrypted via KMS)
                            |
             3. Execute API Calls against Cloud Provider
                            |
             4. Write updated State to S3 (Versioned)
                            |
             5. Release DynamoDB Lock
```

### Backend Configuration Block:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "corp-terraform-state-prod-2026"
    key            = "infrastructure/vpc-core/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-east-1:123456789012:key/mrk-xxxx"
  }
}
```

---

## 3. Remote State Data Source (`terraform_remote_state`)

Allows decoupled root modules (e.g. App Tier) to consume outputs from foundational modules (e.g. VPC Network Tier) without sharing state files:

```hcl
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "corp-terraform-state-prod-2026"
    key    = "infrastructure/vpc-core/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_security_group" "app_sg" {
  name   = "app-tier-sg"
  vpc_id = data.terraform_remote_state.vpc.outputs.vpc_id
}
```
