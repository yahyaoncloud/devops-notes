# Modular Terraform Design & Multi-Environment Workspaces

---

## 1. Enterprise Module Directory Structure

```text
terraform-infrastructure/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── security-groups/
│   │   ├── main.tf
│   │   └── outputs.tf
│   └── ecs-fargate/
│       ├── main.tf
│       └── outputs.tf
└── environments/
    ├── prod/
    │   ├── main.tf
    │   ├── terraform.tfvars
    │   └── backend.tf
    └── dev/
        ├── main.tf
        ├── terraform.tfvars
        └── backend.tf
```

---

## 2. Calling Reusable Child Modules

```hcl
# environments/prod/main.tf
module "network" {
  source = "../../modules/vpc"

  vpc_cidr             = "10.100.0.0/16"
  public_subnet_cidrs  = ["10.100.1.0/24", "10.100.2.0/24"]
  private_subnet_cidrs = ["10.100.10.0/24", "10.100.11.0/24"]
  environment          = "production"
}

module "app_cluster" {
  source = "../../modules/ecs-fargate"

  cluster_name       = "prod-api-cluster"
  vpc_id             = module.network.vpc_id
  private_subnet_ids = module.network.private_subnet_ids
  desired_capacity   = 4
}
```

---

## 3. Workspaces vs Directory-Based Environments

* **Workspaces (`terraform workspace new <name>`):**
  * Uses a single configuration file set but maintains isolated state files (e.g. `env:/staging/terraform.tfstate`).
  * *Best for:* Ephemeral feature branches and staging tiers sharing identical architectures.
* **Directory-Based Separation (Best Practice for Enterprise):**
  * Separate folders for `dev`, `staging`, `prod` with dedicated `backend.tf` and `terraform.tfvars`.
  * *Advantage:* Complete isolation of blast radius, allows testing different module versions in dev before rolling out to prod.
