# Phase 15: Interview Preparation — Lab Exercises

## Lab 1: Timed Design Exercise — Log Aggregation (30 min)
```
Set a 30-minute timer. Practice the full RADIO framework:

  R (5 min): 500 services, 100 GB/day, search < 5s, 30d hot + 1yr cold
  
  A (10 min): Draw the architecture
    Apps (stdout) → Vector DaemonSet → Kafka → Consumers
      → Loki (hot, 30 days) → Grafana
      → S3 Parquet (cold, 1 year) → Athena
      → Alert engine → PagerDuty
  
  D (5 min):
    Log entry: timestamp, service, level, message, trace_id, metadata
    Size: 100 GB/day = 1.2 MB/s average, peak 5 MB/s
    Kafka: 12 partitions, 3 day retention
    Loki: 30 day retention, 3 TB
    S3: 36 TB/year in Parquet (10:1 compression)
  
  I (5 min):
    Query API: GET /logs?service=X&level=error&start=T1&end=T2
    Alert API: POST /alerts (condition, threshold, channel)
    Dashboard: Grafana explore + saved dashboards
  
  O (5 min):
    Scale: Kafka partitions, Loki horizontal scaling
    Cost: Loki vs Elasticsearch ($500 vs $5000/month)
    Reliability: Kafka replication, S3 durability
    Security: log redaction (PII), RBAC on log access
```

## Lab 2: Mock Interview — CI/CD Platform
```
Practice answering out loud (record yourself):

  Interviewer: "Design a CI/CD platform for 100 microservices"

  Your answer should cover:
  1. Requirements clarification (2 min)
     - How many developers? Deploys/day? Languages?
     - SLO for pipeline: < 10 min CI, < 5 min CD
     
  2. Architecture (5 min)
     GitHub → GitHub Actions → ECR → ArgoCD → EKS
     Draw: code repo + manifest repo separation
     
  3. CI details (5 min)
     Stages: lint, test, build, scan, push
     Shared workflow templates (DRY)
     Caching: Docker layers, npm/pip cache
     
  4. CD details (5 min)
     ArgoCD: App-of-Apps pattern
     Canary with Argo Rollouts + analysis
     Environment promotion: staging → production
     
  5. DevOps concerns (5 min)
     Monitoring: DORA metrics (deploy freq, lead time, failure rate, MTTR)
     Security: image scanning, signed images, SBOM
     Cost: ~$500/month (GHA minutes + ECR + ArgoCD)
     
  6. Trade-offs (3 min)
     Jenkins vs GHA: GHA simpler, Jenkins more flexible
     ArgoCD vs Flux: ArgoCD has UI, Flux is lighter
     Monorepo vs polyrepo: polyrepo for independence
```

## Lab 3: Estimation Practice
```
For each scenario, calculate within 2 minutes:

1. Twitter-like service: 500M users, 10% daily active
   - DAU: 50M users
   - Tweets: 50M × 2 tweets/day = 100M tweets/day
   - Read: 50M × 100 reads/day = 5B reads/day = 58K QPS
   - Write: 100M/86400 = 1,160 QPS
   - Storage: 100M × 500B = 50 GB/day = 18 TB/year

2. Instagram-like service: 100M users, 5M uploads/day
   - Storage: 5M × 2MB = 10 TB/day = 3.6 PB/year
   - CDN: 100M users × 50 views × 200KB = 1 PB/day transfer
   - Thumbnail: 5M × 3 sizes × 50KB = 750 GB/day

3. Chat application: 10M users, 1M concurrent
   - WebSocket connections: 1M concurrent
   - Messages: 1M users × 10 msg/min = 10M msg/min = 167K QPS
   - Server: 1 server handles 50K connections → 20 servers
   - Storage: 10M msg/min × 200B = 2 GB/min = 2.8 TB/day
```

## Lab 4: System Design Study Plan
```
Week 1-2: Fundamentals
  ✅ Phase 1-3: Fundamentals, HA, Scalability
  Practice: Estimation exercises daily

Week 3-4: Core Infrastructure
  ✅ Phase 4-6: Load Balancing, Caching, Databases
  Practice: Design URL shortener, rate limiter

Week 5-6: Data & Operations
  ✅ Phase 7-9: Queues, Storage, Monitoring/SRE
  Practice: Design notification system, log aggregation

Week 7-8: Platform & Security
  ✅ Phase 10-12: CI/CD, Containers, IaC
  Practice: Design CI/CD platform, K8s platform

Week 9-10: Advanced & Interview
  ✅ Phase 13-15: Security, Cloud, Interview Prep
  Practice: Full mock interviews (30 min timed)

Daily habits:
  - 1 estimation exercise (2 min)
  - 1 cheat sheet review (5 min)
  - 1 interview Q&A review (10 min)
  - Weekend: 1 full design exercise (30 min)
```