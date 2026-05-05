# Phase 15: Interview Preparation — Cheat Sheet

## RADIO Framework
```
R: Requirements (func + non-func, clarify, don't assume)
A: Architecture (high-level diagram, read/write paths)
D: Data Model (SQL vs NoSQL, entities, size estimation)
I: Interfaces (API design, service communication)
O: Optimize (cache, shard, async, failover, trade-offs)
```

## DevOps Differentiator — Always Mention
```
Deployment: K8s, Helm, ArgoCD, canary/blue-green
Monitoring: Prometheus, SLOs, error budgets, alerting
Scaling:    HPA, cluster autoscaler, caching, sharding
Reliability: Multi-AZ, failover, DR, chaos testing
Security:   mTLS, Vault, IRSA, image scanning
Cost:       Estimation, right-sizing, reserved/spot
Operations: Logs, traces, runbooks, incident response
```

## Estimation Quick Reference
```
1M users, 1 req/day = 12 QPS (peak: 36 QPS)
1M users × 1KB = 1 GB storage
100K QPS × 1KB = 100 MB/s bandwidth
1 server = 1K-5K QPS (dynamic)
1 Redis = 100K QPS, up to ~100 GB
```

## Common DevOps Design Questions
```
Log aggregation pipeline          CI/CD platform
Monitoring/alerting system        Zero-downtime deployment
Secrets management                Multi-tenant K8s platform
Disaster recovery strategy        Cost optimization system
Incident management platform      Feature flag system
```

## Trade-Off Answers Template
```
"It depends on [X]. If [condition A], I'd choose [option 1] because [reason].
If [condition B], I'd go with [option 2] because [reason].
In most cases, I'd default to [option] and adjust based on [metric]."
```