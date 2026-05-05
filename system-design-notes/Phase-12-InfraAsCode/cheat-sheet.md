# Phase 12: Infrastructure as Code — Cheat Sheet

## Terraform Workflow
```
init → plan → apply → destroy
State: S3 + DynamoDB lock (NEVER local in production)
```

## Terraform Key Commands
```
terraform init                  # Initialize
terraform plan -out=plan.out    # Preview changes
terraform apply plan.out        # Apply saved plan
terraform import TYPE.NAME ID   # Import existing resource
terraform state list            # List managed resources
terraform state rm TYPE.NAME    # Remove from state (don't destroy)
terraform output                # Show outputs
```

## State Management
```
Remote backend: S3 + DynamoDB (locking + encryption)
State per component: vpc, eks, app (blast radius reduction)
Drift detection: scheduled terraform plan in CI
```

## Module Structure
```
modules/vpc/         (reusable)
modules/eks/
environments/staging/   (calls modules with staging vars)
environments/production/
```

## Terraform vs Ansible vs Pulumi
```
Terraform:  Provision infra (VPC, EC2, RDS). Declarative HCL.
Ansible:    Configure resources (packages, files). Agentless SSH.
Pulumi:     Provision infra with real languages (TS, Python).
```

## Policy as Code
```
OPA/Gatekeeper:  K8s admission policies
Sentinel:        Terraform Cloud policies
Checkov/tfsec:   Static analysis in CI
```

## Best Practices
```
Pin provider versions           Small state files
Modules for reusability         Variables, no hardcoding
Plan in CI, apply with approval Drift detection alerts
sensitive = true for secrets    Import before recreate
```