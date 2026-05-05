# Phase 3: Scalability Patterns — Cheat Sheet

## Scaling Quick Reference
```
Vertical:    Bigger machine (limit: biggest instance, expensive)
Horizontal:  More machines (requires stateless, linear cost)
```

## Auto-Scaling
```
HPA:          Scale pods on CPU/memory/custom metrics
VPA:          Right-size pod resources (requests/limits)
Cluster AS:   Add/remove nodes based on pending pods
Karpenter:    Smart node provisioning (right-sizes instances)
Predictive:   ML-based, scales before spike
Scheduled:    Cron-based for known events (Black Friday)
```

## Stateless Design
```
Session → Redis          Files → S3
Config → Vault/SSM       Cache → Redis (not in-memory)
Result: Any pod handles any request → easy scaling
```

## Database Scaling
```
Read replicas:    Scale reads (replication lag 10-100ms)
Sharding:         Split data by key across DBs
  Hash:     hash(key) % N      (even distribution)
  Range:    key ranges per shard (easy range queries)
Connection pool:  PgBouncer/ProxySQL (10K→100 connections)
```

## Scaling Numbers
```
Single server:    10K-50K QPS (stateless API)
Redis:            100K+ ops/sec
PostgreSQL:       5K-20K QPS (depends on query)
Kafka broker:     1M+ msgs/sec
Web: 1M DAU ≈ 12 QPS avg, 36 QPS peak (if 1 req/user/day)
```

## Async Pattern
```
API → 202 Accepted → Queue → Workers → Result
Benefits: Lower latency, absorb spikes, independent scaling
Scale workers on: queue depth (SQS ApproximateNumberOfMessages)
```