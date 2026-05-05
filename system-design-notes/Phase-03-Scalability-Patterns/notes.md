# Phase 3: Scalability Patterns — Notes

## 1. Horizontal vs Vertical Scaling

```
Vertical (Scale Up):
  Add CPU/RAM/disk to existing machine
  RDS: db.t3.medium → db.r5.4xlarge
  Limit: biggest instance type, single machine
  Downtime: usually requires restart
  Cost: exponential (2x CPU ≠ 2x price, it's 3-4x)

Horizontal (Scale Out):
  Add more machines
  K8s: replicas: 3 → replicas: 10
  Limit: practically unlimited
  Requires: stateless design, load balancing
  Cost: linear (2x machines ≈ 2x price)
```

## 2. Auto-Scaling

```
Reactive Scaling (based on current metrics):
  K8s HPA: scale pods when CPU > 70%
  AWS ASG: scale instances when CPU > 70%
  Response time: 2-5 minutes (provision + warm-up)
  
  HPA Example:
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    spec:
      minReplicas: 3
      maxReplicas: 50
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 70

Predictive Scaling:
  ML-based: learns traffic patterns, scales BEFORE spike
  AWS Predictive Scaling: analyzes 14 days of data
  Useful for: daily patterns (9 AM spike), weekly (Monday morning)

Scheduled Scaling:
  Cron-based: scale up before known events
  Example: Black Friday — pre-scale to 10x at midnight
  K8s CronHPA or AWS Scheduled Actions

Cluster Autoscaler (nodes):
  Pods pending (can't schedule) → add nodes
  Nodes underutilized → remove nodes (with PDB respect)
  Karpenter (AWS): faster, right-sizes instance types
```

## 3. Stateless Design

```
Why stateless: Any instance can handle any request
  → Easy horizontal scaling
  → Easy replacement (cattle, not pets)
  → Easy deployment (rolling update)

Making services stateless:
  Session state → Redis/Memcached
  File uploads → S3/GCS
  Config → ConfigMaps/Vault/SSM
  Caches → Redis (shared) instead of in-memory (local)
  
  Before (stateful):
    Request → Server A (has session in memory)
    Server A dies → session lost!
  
  After (stateless):
    Request → Any server → reads session from Redis
    Any server dies → others continue serving
```

## 4. Database Scaling Patterns

```
Read Replicas:
  Primary handles writes
  Replicas handle reads (scale out reads)
  Replication lag: 10-100ms
  Good for: read-heavy workloads (10:1 read:write)

Sharding (Horizontal Partitioning):
  Split data across multiple databases by key
  Shard key: user_id, region, tenant_id
  Each shard is independent (own primary + replica)
  
  Strategies:
    Hash-based: shard = hash(user_id) % num_shards
    Range-based: users 1-1M → shard1, 1M-2M → shard2
    Directory: lookup table maps key → shard
  
  Challenges:
    Cross-shard queries (joins) → expensive
    Rebalancing (adding shards) → data migration
    Hot shards → uneven distribution

Connection Pooling:
  Problem: 1000 pods × 10 connections = 10,000 DB connections!
  Solution: PgBouncer / ProxySQL between app and DB
  Pools connections: 10,000 app connections → 100 DB connections
```

## 5. Caching for Scale

```
Cache reduces load on database → enables scaling

Where to cache:
  CDN:           Static assets at edge (CloudFront)
  API Gateway:   Response caching per endpoint
  Application:   Redis/Memcached for query results
  Database:      Query cache, materialized views

Cache hit ratio:
  Target: > 90%
  Low ratio → wrong cache key or wrong data being cached
  
  Monitor: cache_hits / (cache_hits + cache_misses)
```

## 6. Async Processing

```
Synchronous: Client waits for response
  POST /order → validate → charge → ship → response (5 seconds)

Asynchronous: Client gets immediate ack, processing happens later
  POST /order → validate → 202 Accepted → done (100ms)
  Background: queue → charge → ship → notify

Benefits:
  - Lower latency for user
  - Absorb traffic spikes (queue buffers)
  - Independent scaling (workers scale separately)
  - Retry failed tasks automatically

Pattern: API → Message Queue → Workers → Result Store
  Kafka, SQS, RabbitMQ for the queue
  Workers auto-scale based on queue depth
```

---

> **Key insight**: Scale = stateless services + caching + async processing + database sharding