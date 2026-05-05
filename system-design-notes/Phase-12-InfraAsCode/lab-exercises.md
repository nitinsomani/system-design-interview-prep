# Phase 12: Infrastructure as Code — Lab Exercises

## Lab 1: Terraform Remote State Setup
```hcl
# backend.tf — Bootstrap (run once)
resource "aws_s3_bucket" "tfstate" {
  bucket = "mycompany-terraform-state"
  lifecycle { prevent_destroy = true }
}

resource "aws_s3_bucket_versioning" "tfstate" {
  bucket = aws_s3_bucket.tfstate.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "tfstate" {
  bucket = aws_s3_bucket.tfstate.id
  rule {
    apply_server_side_encryption_by_default { sse_algorithm = "aws:kms" }
  }
}

resource "aws_dynamodb_table" "tflock" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute { name = "LockID"; type = "S" }
}

# Then configure backend in each project:
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "production/vpc/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

## Lab 2: Reusable Module
```hcl
# modules/vpc/main.tf
variable "name" { type = string }
variable "cidr" { type = string; default = "10.0.0.0/16" }
variable "azs"  { type = list(string) }

resource "aws_vpc" "main" {
  cidr_block           = var.cidr
  enable_dns_hostnames = true
  tags = { Name = var.name }
}

resource "aws_subnet" "public" {
  count             = length(var.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.cidr, 8, count.index)
  availability_zone = var.azs[count.index]
  tags = { Name = "${var.name}-public-${var.azs[count.index]}" }
}

output "vpc_id"     { value = aws_vpc.main.id }
output "subnet_ids" { value = aws_subnet.public[*].id }

# environments/production/main.tf
module "vpc" {
  source = "../../modules/vpc"
  name   = "prod-vpc"
  cidr   = "10.0.0.0/16"
  azs    = ["us-east-1a", "us-east-1b", "us-east-1c"]
}
```

## Lab 3: CI/CD Pipeline for Terraform
```yaml
# .github/workflows/terraform.yml
name: Terraform
on:
  pull_request: { paths: ['infra/**'] }
  push: { branches: [main], paths: ['infra/**'] }

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3

      - name: Format Check
        run: terraform fmt -check -recursive
        working-directory: infra/

      - name: Init
        run: terraform init
        working-directory: infra/environments/production

      - name: Validate
        run: terraform validate
        working-directory: infra/environments/production

      - name: Security Scan
        uses: aquasecurity/tfsec-action@v1.0.0
        with: { working_directory: infra/ }

      - name: Plan
        run: terraform plan -out=plan.out
        working-directory: infra/environments/production

  apply:
    if: github.ref == 'refs/heads/main'
    needs: plan
    runs-on: ubuntu-latest
    environment: production  # requires manual approval
    steps:
      - name: Apply
        run: terraform apply plan.out
        working-directory: infra/environments/production
```

## Lab 4: IaC Design Exercise
```
Scenario: Design IaC structure for a platform with:
  - 3 environments (dev, staging, prod)
  - VPC, EKS, RDS, ElastiCache, S3
  - 5 microservices

  Structure:
  infra/
    modules/
      vpc/            (VPC, subnets, NAT, IGW)
      eks/            (EKS cluster, node groups, IRSA)
      rds/            (PostgreSQL, parameter groups, backups)
      elasticache/    (Redis cluster)
      s3/             (Buckets with lifecycle)
    environments/
      dev/
        main.tf       (smaller instances, single AZ)
        terraform.tfvars
      staging/
        main.tf       (production-like, multi-AZ)
        terraform.tfvars
      production/
        main.tf       (full scale, multi-AZ, DR)
        terraform.tfvars

  State files (per env, per component):
    s3://tfstate/dev/vpc/terraform.tfstate
    s3://tfstate/dev/eks/terraform.tfstate
    s3://tfstate/prod/vpc/terraform.tfstate
    s3://tfstate/prod/eks/terraform.tfstate
    → 15 state files (3 envs × 5 components)
    → Blast radius: bad apply to RDS won't affect EKS
```