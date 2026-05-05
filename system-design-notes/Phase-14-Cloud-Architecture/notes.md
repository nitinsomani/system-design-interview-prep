# Phase 14: Cloud Architecture — Notes

## 1. Well-Architected Framework (AWS)

```
6 Pillars:
  1. Operational Excellence: Automate, IaC, runbooks, postmortems
  2. Security: IAM, encryption, least privilege, detective controls
  3. Reliability: Multi-AZ, auto-scaling, backup, chaos testing
  4. Performance Efficiency: Right-size, caching, CDN, serverless
  5. Cost Optimization: Reserved/Spot, right-size, lifecycle policies
  6. Sustainability: Efficient workloads, minimize waste
```

## 2. Multi-AZ & Multi-Region

```
Multi-AZ (standard for production):
  - Resources in 2-3 AZs within one region
  - AZ failure → traffic shifts automatically
  - RDS Multi-AZ: synchronous standby, automatic failover
  - EKS: nodes across AZs, topology spread constraints
  - Cost: ~1.5x (data transfer between AZs)

Multi-Region (for global or DR):
  Active-Passive:
    Primary region serves traffic
    Secondary: standby, receives replicated data
    Failover: DNS switch (Route 53 health check → failover)
    RPO: replication lag (seconds to minutes)
    RTO: DNS propagation + warmup (minutes to hours)

  Active-Active:
    Both regions serve traffic simultaneously
    Global database: Aurora Global, DynamoDB Global Tables
    Route 53 latency-based routing
    Conflict resolution needed for writes
    RPO: ~0, RTO: ~0 (already serving)
    Cost: 2x+, complexity: high
```

## 3. Serverless Architecture

```
Services:
  Compute:   Lambda, Fargate
  API:       API Gateway
  Storage:   S3, DynamoDB
  Queue:     SQS, EventBridge
  Orchestration: Step Functions

Lambda:
  Max execution: 15 minutes
  Memory: 128MB - 10GB
  Cold start: 100ms-5s (depends on runtime, VPC)
  Pricing: per invocation + duration (very cheap at low scale)
  
  Good for: Event processing, webhooks, scheduled tasks, glue logic
  Bad for: Long-running, high-throughput, stateful workloads

Fargate:
  Serverless containers (no EC2 management)
  ECS or EKS mode
  Good for: Microservices that need container flexibility
  More expensive than EC2 at scale (~20-30% premium)

Serverless vs Containers decision:
  < 1M requests/month → Lambda (cheapest)
  1M-100M requests/month → Fargate or EKS
  > 100M requests/month → EKS on EC2 (most cost-effective)
  Stateful / long-running → EKS
  Event-driven / sporadic → Lambda
```

## 4. Cost Optimization

```
Compute:
  Reserved Instances: 1-3 year commitment → 30-60% savings
  Savings Plans: flexible commitment → 20-40% savings
  Spot Instances: up to 90% off, can be interrupted
    Use for: CI/CD runners, batch jobs, stateless workers
    Not for: databases, stateful services

  Right-sizing:
    Monitor CPU/memory → downsize over-provisioned instances
    Tools: AWS Compute Optimizer, Kubecost
    Schedule: review monthly

Storage:
  S3 lifecycle policies (Standard → IA → Glacier)
  Delete unused EBS volumes and snapshots
  Use gp3 over gp2 (20% cheaper, better performance)

Network:
  VPC endpoints (avoid NAT Gateway data charges)
  CloudFront for S3 (cheaper than direct S3 transfer)
  Compress responses (reduce data transfer)

Monitoring:
  AWS Cost Explorer + budgets + anomaly detection
  Kubecost for K8s cost allocation per team/namespace
  Tag everything for cost attribution
```

## 5. Hybrid & Multi-Cloud

```
Hybrid Cloud (on-prem + cloud):
  Connectivity: AWS Direct Connect, VPN
  Services: AWS Outposts, Azure Arc, Anthos
  Use cases: data sovereignty, legacy systems, gradual migration

Multi-Cloud:
  Why: Avoid vendor lock-in, compliance, best-of-breed
  Challenges: Different APIs, networking, IAM, cost models
  Approach: Abstract with Terraform, K8s (runs anywhere)
  
  Reality: Most companies are primarily on one cloud
    Multi-cloud usually means "cloud + SaaS tools"
    True multi-cloud (same app on AWS + GCP) is rare and expensive

Cloud-agnostic tools:
  IaC:          Terraform, Pulumi
  Containers:   Kubernetes (EKS, GKE, AKS)
  Monitoring:   Prometheus, Grafana
  CI/CD:        GitHub Actions, GitLab CI
  Service Mesh: Istio
```

---

> **DevOps focus**: Multi-AZ design, DR strategy, cost optimization, serverless vs containers decision, cloud-agnostic tooling.