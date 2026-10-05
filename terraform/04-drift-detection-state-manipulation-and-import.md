# Terraform Drift Detection, State Manipulation & Import Workflows

---

## 1. Detecting & Reconciling Infrastructure Drift

Infrastructure drift occurs when manual changes (Console/CLI) bypass Terraform:

```bash
# Refresh state against real cloud environment and show drift
terraform plan -refresh-only

# Automated CI/CD Drift Detection (Returns exit code 2 if drift exists)
terraform plan -detailed-exitcode -no-color

# Overwrite out-of-band changes to restore declared HCL state
terraform apply -auto-approve
```

---

## 2. Importing Existing Resources (`import` Block vs CLI)

### Modern HCL `import` Block (Terraform 1.5+):
```hcl
import {
  to = aws_security_group.legacy_firewall
  id = "sg-0123456789abcdef0"
}

resource "aws_security_group" "legacy_firewall" {
  name        = "legacy-fw-rules"
  description = "Imported legacy perimeter rules"
  vpc_id      = "vpc-01a2b3c4d5e6f7"
  # Attributes can be auto-generated via: terraform plan -generate-config-out=generated.tf
}
```

---

## 3. Essential State Manipulation Commands

| Command | Action & Purpose |
| :--- | :--- |
| `terraform state list` | Lists all tracked resource addresses in the current state file. |
| `terraform state show <addr>` | Displays full stored attributes for a specific resource. |
| `terraform state mv <src> <dst>` | Renames or moves a resource inside state without destroying and recreating the cloud resource (e.g. moving a resource into a module). |
| `terraform state rm <addr>` | Removes resource from state tracking without deleting the actual cloud asset. |
| `terraform state pull > state.json`| Fetches raw remote state JSON for inspection or backup. |
| `terraform force-unlock <LOCK_ID>`| Manually removes a stuck DynamoDB lock ID after an aborted run. |
