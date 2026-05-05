# Phase 14: Cloud Architecture — Interview Q&A

## Q1: How do you design for high availability in AWS?
**Answer:** Multi-AZ baseline: 1) **Compute**: EKS nodes across 3 AZs with topology spread constraints + PDB. Auto-scaling group spans AZs. 2) **Database**: RDS Multi-AZ (synchronous standby, auto-failover <60s). ElastiCache Multi-AZ with auto-failover. 3) **Load balancing**: ALB distributes across AZs, health checks remove unhealthy targets. 4) **Storage**: S3 (11 nines durability, auto-replicated). EBS snapshots cross-AZ. 5) **DNS**: Route 53 health checks + failover routing. Result: single AZ failure = no user impact. For 99.99%+: add multi-region active-passive with Route 53 failover.

## Q2: When would you use serverless vs containers?
**Answer:** **Serverless (Lambda/Fargate)**: 1) Low/sporadic traffic (<1M req/month) — pay only for use. 2) Event-driven (S3 events, SQS, API webhooks). 3) Short-lived tasks (<15 min). 4) Small team — less operational overhead. **Containers (EKS)**: 1) High traffic (>100M req/month) — EC2-backed is cheapest. 2) Long-running processes. 3) Stateful workloads. 4) Need full control over runtime. 5) Complex networking (service mesh). Fargate = middle ground: container flexibility without EC2 management, but ~20-30% premium. Most platforms: EKS for core services + Lambda for glue/events.

## Q3: How do you optimize cloud costs?
**Answer:** Three categories: 1) **Right-size**: analyze CPU/memory utilization (Compute Optimizer, Kubecost). Downsize over-provisioned instances. Monthly review. 2) **Pricing models**: Reserved Instances or Savings Plans for baseline (30-60% off). Spot for fault-tolerant workloads (CI runners, batch). 3) **Architecture**: S3 lifecycle policies (save 60-80% on storage). VPC endpoints (avoid NAT data charges $0.045/GB). CloudFront for S3 delivery. Delete unused resources (EBS, EIPs, old snapshots). Tag everything → Cost Explorer → per-team allocation. Set budget alerts. Typical savings: 30-50% from initial unoptimized spend.

## Q4: How do you approach multi-region architecture?
**Answer:** First question: do you actually need it? Multi-AZ handles most availability needs. Multi-region reasons: 1) Global users (latency), 2) Regulatory (data residency), 3) Business continuity (region-level DR). If needed: start with **active-passive** — primary region serves all traffic, secondary has replicated data + minimal compute (warm standby). Failover via Route 53 health check. For global latency: **active-active** with DynamoDB Global Tables or Aurora Global Database. Challenge: conflict resolution on concurrent writes. Cost: 2x+ infrastructure, significant operational complexity. Most companies: multi-AZ + cross-region backups (not full active-active).

## Q5: How do you handle cloud vendor lock-in?
**Answer:** Pragmatic approach: 1) Use cloud-native services where the value is high (RDS, S3, CloudFront) — don't over-abstract. 2) Use portable tools for compute and deployment: K8s (runs on any cloud), Terraform (multi-provider), Prometheus/Grafana (OSS). 3) Avoid deep dependency on proprietary services where alternatives exist (Step Functions → Temporal, SNS → Kafka). 4) Data portability: standard formats (Parquet, JSON), export tools. Full portability is expensive and rarely needed. Real risk mitigation: ensure data is exportable + core compute is on K8s.

## Rapid-Fire
- **AZ vs Region?** → AZ: isolated datacenter within region. Region: geographic area with 3+ AZs.
- **Direct Connect?** → Dedicated network from on-prem to AWS (not over internet)
- **NAT Gateway cost trap?** → $0.045/GB data processing. Use VPC endpoints for AWS services.
- **Savings Plans vs Reserved?** → Savings Plans: flexible across instance types. Reserved: specific instance.
- **AWS Outposts?** → AWS hardware in your datacenter (hybrid cloud)