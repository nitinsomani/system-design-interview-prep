# Phase 12: Infrastructure as Code — Notes

## 1. IaC Principles

```
Declarative:   Describe desired state, tool converges (Terraform, K8s)
Imperative:    Describe steps to reach state (scripts, Ansible tasks)

Benefits:
  - Version controlled (Git = audit trail)
  - Repeatable (same code → same infra)
  - Reviewable (PR-based changes)
  - Testable (plan, validate, lint)
  - Self-documenting (code IS the documentation)

Idempotency:
  Running IaC twice → same result (no duplicate resources)
  Critical for reliability and CI/CD automation
```

## 2. Terraform

```
Core workflow:
  terraform init    → download providers, initialize backend
  terraform plan    → show what will change (diff)
  terraform apply   → execute changes
  terraform destroy → tear down all resources

Key concepts:
  Provider:   AWS, GCP, Azure, K8s (API interface)
  Resource:   Infrastructure object (aws_instance, aws_s3_bucket)
  Data source: Read existing infrastructure
  Module:     Reusable group of resources
  State:      JSON file mapping config to real resources
  Backend:    Where state is stored (S3 + DynamoDB for locking)

State management:
  Local state: terraform.tfstate (NEVER commit to Git)
  Remote state: S3 bucket + DynamoDB lock table
  State locking: prevents concurrent modifications
  
  terraform {
    backend "s3" {
      bucket         = "mycompany-tfstate"
      key            = "prod/vpc/terraform.tfstate"
      region         = "us-east-1"
      dynamodb_table = "terraform-locks"
      encrypt        = true
    }
  }

Module structure:
  modules/
    vpc/
      main.tf
      variables.tf
      outputs.tf
    eks/
      main.tf
      variables.tf
      outputs.tf
  environments/
    staging/
      main.tf      (calls modules with staging vars)
    production/
      main.tf      (calls modules with prod vars)
```

## 3. Terraform Best Practices

```
1. Remote state with locking (S3 + DynamoDB)
2. Small, focused state files (per service/component)
3. Modules for reusability (DRY)
4. Use workspaces OR directory structure for environments
5. Pin provider versions (required_providers)
6. Use terraform plan in CI, apply requires approval
7. Import existing resources (terraform import)
8. Drift detection: plan in CI, alert if drift detected
9. Use variables and locals, avoid hardcoded values
10. Sensitive variables: mark sensitive = true
```

## 4. Ansible

```
Purpose: Configuration management, application deployment
  Agentless: SSH to target machines
  Idempotent: modules check current state before acting

Key concepts:
  Inventory:   List of hosts (static or dynamic)
  Playbook:    YAML file with ordered tasks
  Role:        Reusable group of tasks + handlers + vars
  Module:      Unit of work (apt, copy, template, docker_container)
  Handler:     Triggered by notify (restart service after config change)

Terraform vs Ansible:
  Terraform:  Infrastructure provisioning (VPC, EC2, RDS)
  Ansible:    Configuration of provisioned resources (install packages, configure)
  Together:   Terraform creates infra → Ansible configures it

When Ansible alone:
  - Legacy infrastructure (bare metal, VMs)
  - Configuration management (packages, files, services)
  - Application deployment (non-containerized)
```

## 5. Pulumi

```
IaC using real programming languages (TypeScript, Python, Go)

Advantages over Terraform:
  - Real language: loops, conditionals, functions, testing
  - IDE support: autocomplete, type checking
  - Testing: unit test infrastructure code
  - Abstraction: create components as classes

When to use:
  - Team already strong in TypeScript/Python
  - Complex logic in infrastructure (conditional resources)
  - Want unit testing for IaC
  - Startup / greenfield project
```

## 6. Policy as Code

```
Enforce rules on infrastructure changes:

OPA (Open Policy Agent) / Gatekeeper:
  K8s admission control
  "No pods without resource limits"
  "No public S3 buckets"
  "Images must come from approved registry"

Sentinel (HashiCorp):
  Terraform Cloud/Enterprise policy
  "EC2 instances must be t3.medium or smaller"
  "All resources must have cost-center tag"

Checkov / tfsec:
  Static analysis for Terraform
  Scan HCL for security misconfigurations
  Run in CI pipeline
```

---

> **DevOps focus**: Terraform state management, module design, drift detection, CI/CD integration, policy enforcement.