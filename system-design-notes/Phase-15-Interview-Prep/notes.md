# Phase 15: Interview Preparation — Notes

## 1. System Design Interview Framework (RADIO)

```
R - Requirements (5 min)
  Functional: What does the system do?
  Non-functional: Scale, latency, availability, consistency
  Constraints: Budget, timeline, team size
  Ask clarifying questions — don't assume

A - Architecture (10 min)
  High-level components: API, services, DB, cache, queue
  Draw the diagram: Client → LB → API → Service → DB
  Identify data flow: read path vs write path

D - Data Model (5 min)
  Choose database (SQL vs NoSQL)
  Define key entities and relationships
  Estimate data size and growth

I - Interfaces (5 min)
  API design: endpoints, request/response
  Service-to-service communication
  External integrations

O - Optimize (10 min)
  Scale: caching, sharding, async processing
  Reliability: replication, failover, monitoring
  Trade-offs: consistency vs availability, cost vs performance
```

## 2. DevOps-Specific Design Considerations

```
Always address (even if not asked):
  1. Deployment: How is this deployed? (K8s, Helm, ArgoCD)
  2. Monitoring: How do you know it's healthy? (Prometheus, SLOs)
  3. Scaling: How does it handle 10x traffic? (HPA, auto-scaling)
  4. Reliability: What happens when X fails? (failover, DR)
  5. Security: How is data protected? (encryption, IAM, mTLS)
  6. Cost: What does this cost at scale? (estimation)
  7. Operations: How do you debug issues? (logs, traces, runbooks)

This is what differentiates a DevOps answer from a SWE answer.
```

## 3. Common DevOps System Design Questions

```
Design a log aggregation pipeline
Design a CI/CD platform
Design a monitoring/alerting system
Design a deployment pipeline with zero-downtime
Design a secrets management system
Design a multi-tenant Kubernetes platform
Design a disaster recovery strategy
Design a cost optimization system
Design an incident management platform
Design a feature flag system
```

## 4. Design Exercise: Log Aggregation Pipeline

```
Requirements:
  - Collect logs from 1000 microservices
  - 50 GB/day of logs
  - Search within 5 seconds
  - Retain 30 days hot, 1 year cold
  - Alert on error patterns

Architecture:
  ┌─────────┐    ┌─────────┐    ┌───────┐    ┌──────────┐
  │ Apps    │ →  │ Vector  │ →  │ Kafka │ →  │ Consumer │
  │ (stdout)│    │ DaemonSet│   │       │    │          │
  └─────────┘    └─────────┘    └───────┘    └────┬─────┘
                                                   │
                                    ┌──────────────┼──────────────┐
                                    ↓              ↓              ↓
                              ┌──────────┐  ┌───────────┐  ┌──────────┐
                              │ Loki/ES  │  │ S3        │  │ Alert    │
                              │ (hot)    │  │ (cold)    │  │ Engine   │
                              └──────────┘  └───────────┘  └──────────┘

Components:
  Collection: Vector DaemonSet (lightweight, fast)
  Buffer: Kafka (handle spikes, decouple)
  Hot storage: Loki or Elasticsearch (searchable, 30 days)
  Cold storage: S3 in Parquet format (1 year, cheap)
  Query: Grafana (Loki) or Kibana (ES)
  Alerting: Prometheus rules on log patterns

Scaling:
  Vector: 1 per node (DaemonSet)
  Kafka: 12 partitions, 3 brokers
  Loki: 3 ingesters, 2 queriers
  S3: unlimited

Cost:
  Loki (30 days): ~$500/month
  S3 (1 year): ~$100/month
  Kafka: ~$300/month
  vs Datadog: ~$5,000/month (at this scale)
```

## 5. Design Exercise: CI/CD Platform

```
Requirements:
  - 50 microservices, 200 developers
  - 100 deploys/day to production
  - Zero-downtime deployments
  - Automated rollback on failures

Architecture:
  GitHub PR → GitHub Actions CI → ECR → ArgoCD → EKS

  CI Pipeline (per service):
    1. Lint + Static Analysis (30s)
    2. Unit Tests (2 min)
    3. Build multi-arch image (3 min)
    4. Security scan: Trivy + Semgrep (1 min)
    5. Push to ECR with Git SHA tag (30s)
    6. Update manifest repo image tag (30s)
    Total: ~7 min

  CD Pipeline (ArgoCD):
    1. Detect manifest change
    2. Sync to staging (auto)
    3. Run integration tests
    4. Manual approval for production
    5. Canary rollout (10% → 50% → 100%)
    6. Analysis template: error rate, P99 latency
    7. Auto-rollback if analysis fails

  Key decisions:
    - Monorepo vs polyrepo: polyrepo (independent pipelines)
    - Image tagging: Git SHA (immutable, traceable)
    - Secret management: External Secrets Operator + Vault
    - Cost: GitHub Actions + ECR + ArgoCD ≈ $500/month
```

## 6. Estimation Cheat Sheet

```
Quick numbers for interviews:
  QPS:
    1 request/user/day → 1M users → 12 QPS
    10 requests/user/day → 1M users → 120 QPS
    Peak: 2-3x average

  Storage:
    1M users × 1KB profile = 1 GB
    1M posts/day × 500B = 500 MB/day = 180 GB/year
    1M images/day × 500KB = 500 GB/day

  Bandwidth:
    100K QPS × 1KB response = 100 MB/s = 800 Mbps

  Server capacity:
    1 web server: ~10K-50K QPS (static/cached)
    1 web server: ~1K-5K QPS (dynamic, DB-backed)
    1 DB server: ~5K-10K QPS (indexed queries)

  Memory:
    1M entries × 1KB = 1 GB (fits in one Redis)
    100M entries × 1KB = 100 GB (Redis cluster)
```

---

> **DevOps focus**: Always address deployment, monitoring, scaling, reliability, security, and cost in your design answers.