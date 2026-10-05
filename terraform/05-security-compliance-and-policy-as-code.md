# Policy as Code (PaC) & Static Security Analysis
### Checkov • tfsec • Open Policy Agent (OPA) / Rego

Static analysis tools catch cloud misconfigurations (unencrypted S3 buckets, open SSH `0.0.0.0/0`, missing logging) before deployment into production.

---

## 1. Static Analysis Tool Matrix

| Tool | Engine | Focus | Integration Point |
| :--- | :--- | :--- | :--- |
| **Checkov** | Python AST & Graph Framework | 1,000+ CIS benchmarks across AWS, Azure, GCP, K8s, Dockerfile, Terraform. | Pre-commit hooks & GitHub Actions CI/CD. |
| **tfsec / Trivy** | Go AST Parser | Ultra-fast local security scanner targeting Terraform code. | Local developer CLI & pre-commit. |
| **Open Policy Agent (OPA)** | Rego Query Language | Custom organizational compliance guardrails evaluated against Terraform JSON plan. | CI/CD Merge Gates. |

---

## 2. Checkov CI/CD Pipeline Scanning Example

```bash
# Scan local Terraform directory against CIS Benchmarks
checkov -d ./terraform-infrastructure --framework terraform

# Run soft fail (exit code 0) for non-blocking audits
checkov -d . --soft-fail

# Export results to SARIF for GitHub Security tab integration
checkov -d . -o sarif > checkov-results.sarif
```

---

## 3. Custom Policy as Code with OPA / Rego

**Rule:** Ensure no Security Group allows inbound SSH (Port 22) from `0.0.0.0/0`:

```rego
package terraform.security

default allow = true

deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_security_group_rule"
    resource.change.after.type == "ingress"
    resource.change.after.from_port <= 22
    resource.change.after.to_port >= 22
    cidr := resource.change.after.cidr_blocks[_]
    cidr == "0.0.0.0/0"
    msg := sprintf("CRITICAL: SG Rule '%v' allows unrestricted SSH from 0.0.0.0/0", [resource.address])
}
```
