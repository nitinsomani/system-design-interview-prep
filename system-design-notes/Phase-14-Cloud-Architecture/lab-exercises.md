# Phase 14: Cloud Architecture — Lab Exercises

## Lab 1: Multi-AZ EKS Design
```bash
# EKS node group across 3 AZs
# eksctl create cluster --name prod --region us-east-1 \
#   --zones us-east-1a,us-east-1b,us-east-1c \
#   --node-type t3.large --nodes-min 2 --nodes-max 10

# Ensure pods spread across AZs
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata: { name: api }
spec:
  replicas: 6
  selector: { matchLabels: { app: api } }
  template:
    metadata: { labels: { app: api } }
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector: { matchLabels: { app: api } }
      containers:
        - name: api
          image: myapp:v1
          resources:
            requests: { cpu: 200m, memory: 256Mi }
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata: { name: api-pdb }
spec:
  minAvailable: 4
  selector: { matchLabels: { app: api } }
EOF
# Result: 2 pods per AZ, min 4 always available
```

## Lab 2: Cost Analysis Exercise
```
Current monthly bill: $50,000
  EC2: $25,000 (50 instances, all on-demand)
  RDS: $8,000 (2 Multi-AZ instances)
  S3: $5,000 (100 TB, all Standard)
  NAT Gateway: $7,000 (data transfer)
  Other: $5,000

  Optimization plan:
  1. EC2 → Savings Plan for baseline 30 instances: $15,000 → $9,000
     Spot for CI/CD runners (10 instances): $2,500 → $500
     Right-size remaining 10: $5,000 → $3,000
     EC2 total: $25,000 → $12,500 (50% savings)

  2. S3 → Lifecycle policy (30d → IA, 90d → Glacier):
     $5,000 → $2,000 (60% savings)

  3. NAT → VPC endpoints for S3, ECR, CloudWatch:
     $7,000 → $3,000 (57% savings)

  4. RDS → Reserved Instance (1yr): $8,000 → $5,000

  New total: $27,500 (45% savings = $22,500/month saved)
```

## Lab 3: DR Strategy Design
```
Scenario: E-commerce platform, RPO < 5 min, RTO < 1 hour

  Primary: us-east-1 (active)
  DR: us-west-2 (passive warm standby)

  Data replication:
    RDS: Cross-region read replica (async, lag < 1 min)
    S3: Cross-region replication (real-time)
    Redis: Global Datastore (async, lag < 1s)

  DR region (warm standby):
    EKS: 2-node cluster (scale up on failover)
    RDS: Promoted from read replica (5 min)
    ALB: Pre-configured, no traffic

  Failover trigger:
    Route 53 health check fails on primary → automatic DNS failover
    OR manual: promote RDS replica, scale EKS, update DNS

  Failover steps (automated):
    1. Route 53 detects primary unhealthy (30s)
    2. DNS failover to DR ALB (60s propagation)
    3. EKS auto-scales from 2 → 10 nodes (5 min)
    4. RDS replica promoted to primary (5 min)
    Total RTO: ~10 min (within 1 hour requirement)

  Testing: Quarterly DR drill — failover + failback
```

## Lab 4: Architecture Decision Exercise
```
For each scenario, choose the architecture:

1. Startup MVP (100 users, 3 developers)
   → Serverless: Lambda + API Gateway + DynamoDB + S3
   Reason: Zero ops, pay-per-use, cheapest at low scale

2. SaaS platform (10K users, 500 QPS)
   → EKS on Fargate, RDS PostgreSQL, ElastiCache Redis
   Reason: Moderate scale, need flexibility, no EC2 mgmt

3. High-traffic marketplace (1M users, 50K QPS)
   → EKS on EC2 (Reserved), RDS Aurora, ElastiCache, CloudFront
   Reason: Cost-effective at scale, full control

4. Global real-time gaming (5M users, worldwide)
   → Multi-region active-active, DynamoDB Global Tables
     CloudFront + Lambda@Edge, WebSocket via API Gateway
   Reason: Ultra-low latency globally, eventual consistency OK
```