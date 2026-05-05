# Phase 3: Scalability Patterns — Interview Q&A

## Q1: How would you scale a service from 100 to 100K requests per second?
**Answer:** Phase approach: 1) Stateless design (externalize sessions/state to Redis). 2) Horizontal scaling (HPA, pods across AZs). 3) Caching (Redis for hot data, CDN for static). 4) Read replicas for DB reads. 5) Async processing (offload writes to queue + workers). 6) Database sharding if single DB is bottleneck. 7) Connection pooling (PgBouncer). Each step handles ~10x growth before needing the next.

## Q2: What's the difference between HPA, VPA, and Cluster Autoscaler?
**Answer:** HPA: scales pod count (replicas) based on metrics (CPU, custom). VPA: adjusts pod resource requests/limits (right-sizing). Cluster Autoscaler: adds/removes nodes when pods can't be scheduled. They work together: HPA adds pods → no room → CA adds nodes. VPA is usually for non-HPA workloads (can conflict). Karpenter replaces CA with smarter provisioning.

## Q3: How do you handle database connection limits at scale?
**Answer:** Problem: 500 pods × 20 connections = 10,000 connections; PostgreSQL handles ~5000 max. Solution: PgBouncer (connection pooler) sits between app and DB — 10,000 app connections multiplexed to 200 DB connections. Deploy PgBouncer as sidecar or separate service. Also: tune pool size per pod, use transaction-mode pooling, monitor `active` vs `idle` connections.

## Q4: Explain sharding strategies and their tradeoffs.
**Answer:** Hash-based: `shard = hash(key) % N` — even distribution, but adding shards requires rehashing (use consistent hashing). Range-based: key ranges per shard — good for range queries, but hot spots if ranges are uneven. Directory/lookup: mapping table — flexible but adds latency and SPOF. Key choice is critical: `user_id` keeps user's data together; `order_id` distributes evenly but cross-user queries need scatter-gather.

## Q5: When would you use async processing vs synchronous?
**Answer:** Sync: user needs immediate response (login, search, read). Async: user can wait (email, report generation, payment processing). Pattern: accept request synchronously (202 Accepted), process asynchronously (queue → worker), notify on completion (webhook/polling). Benefits: lower user-facing latency, absorb traffic spikes (queue buffers), independent scaling of API vs workers.

## Rapid-Fire
- **Stateless service benefit?** → Any instance handles any request → easy scaling
- **HPA metric for queue workers?** → Queue depth (SQS ApproximateNumberOfMessages)
- **Connection pooler for PostgreSQL?** → PgBouncer (transaction mode)
- **Consistent hashing solves?** → Adding/removing shards with minimal data movement
- **Thundering herd?** → Cache expires → all requests hit DB simultaneously
- **Backpressure?** → Slow down producers when consumers can't keep up
- **Karpenter vs Cluster Autoscaler?** → Karpenter: faster, right-sizes instance types