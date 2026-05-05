# Phase 12: Infrastructure as Code — Interview Q&A

## Q1: How do you manage Terraform state in a team?
**Answer:** Remote backend with S3 + DynamoDB: S3 stores state files (encrypted, versioned), DynamoDB provides state locking (prevents concurrent applies). Structure: separate state files per component (vpc, eks, app) to reduce blast radius — a bad apply on the app state won't affect networking. Access: IAM policies restrict who can read/modify state. CI/CD: `terraform plan` on PR (automated), `terraform apply` on merge with approval. State contains sensitive data → treat S3 bucket as confidential (encryption, access logs).

## Q2: How do you handle infrastructure drift?
**Answer:** Drift = actual infra differs from Terraform state/code. Detection: scheduled `terraform plan` in CI (e.g., daily cron) — if plan shows changes, alert the team. Prevention: 1) All changes through Terraform (never manual console changes). 2) RBAC: limit console write access. 3) SCPs to prevent resource creation outside IaC. Remediation: review drift → either update code to match reality (intended change) or `terraform apply` to fix drift (unintended). Tools: Driftctl for comprehensive drift detection including unmanaged resources.

## Q3: How do you structure Terraform for multiple environments?
**Answer:** Two approaches: 1) **Directory-based**: `environments/staging/main.tf` and `environments/production/main.tf` — each calls shared modules with different variables. Pros: clear separation, independent state. 2) **Workspaces**: same code, `terraform workspace select staging`. Cons: easy to accidentally apply to wrong workspace. I prefer directory-based: explicit, separate state files, separate CI/CD pipelines. Shared logic in modules (`modules/vpc/`, `modules/eks/`). Environment-specific values in tfvars files.

## Q4: Terraform vs Pulumi — when to use which?
**Answer:** **Terraform**: industry standard, large ecosystem, HCL is simple enough for most infra. Best when: team has mixed skill levels, need broad provider support, hiring (more people know TF). **Pulumi**: real programming languages (TypeScript, Python), better for complex logic (conditionals, loops, component abstractions), unit testable. Best when: team is strong in TypeScript/Python, need complex infrastructure logic, want IDE autocomplete and type safety. Both: declarative state management, plan/preview, similar provider coverage. Default: Terraform unless you have a specific reason for Pulumi.

## Q5: How do you implement a CI/CD pipeline for Terraform?
**Answer:** 1) **PR opened**: `terraform fmt -check` + `terraform validate` + `terraform plan` → post plan output as PR comment. Checkov/tfsec security scan. 2) **PR approved + merged**: `terraform apply` with saved plan file. Require manual approval for production. 3) **Drift detection**: daily cron runs `terraform plan` → alert if changes detected. 4) **State management**: each environment has its own pipeline and state. 5) **Secrets**: Terraform variables via CI secrets (never in code). Tools: GitHub Actions, Atlantis (automated plan/apply on PRs), Terraform Cloud.

## Rapid-Fire
- **terraform taint?** → Mark resource for recreation on next apply (deprecated, use -replace)
- **terraform import?** → Bring existing resource under Terraform management
- **Count vs for_each?** → for_each preferred (stable keys, no index shift issues)
- **Data source?** → Read existing infrastructure without managing it
- **Lifecycle prevent_destroy?** → Prevent accidental deletion of critical resources
- **terraform state mv?** → Rename/move resources in state without destroy/recreate