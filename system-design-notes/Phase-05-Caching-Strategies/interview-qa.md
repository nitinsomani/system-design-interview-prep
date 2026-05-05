# Phase 5: Caching Strategies — Interview Q&A

## Q1: How do you decide what to cache and what caching pattern to use?
**Answer:** Cache data that is: read frequently, expensive to compute, tolerant of slight staleness. Pattern selection: Cache-aside for most cases (read-heavy, independent cache). Write-through when you need strong consistency. Write-behind for write-heavy workloads where eventual consistency is OK (analytics, counters). Read-through when cache library supports it and you want simpler app code. Key metric: if cache hit ratio < 80%, you're caching the wrong things.

## Q2: How do you handle cache invalidation in a microservices architecture?
**Answer:** Three approaches: 1) TTL-based: simple, accept staleness up to TTL (good for 90% of cases). 2) Event-driven: service publishes event on data change → consumer deletes cache key. Implement via Kafka/SNS + Debezium (CDC). 3) Versioned keys: include version in key (user:123:v5), bump on write, old keys expire via TTL. For cross-service cache: prefer events over direct cache manipulation (coupling). Never have Service B delete Service A's cache directly.

## Q3: Explain the thundering herd problem and how to prevent it.
**Answer:** When a popular cache key expires, hundreds of concurrent requests all miss cache and hit the DB simultaneously, potentially overloading it. Solutions: 1) Singleflight/request coalescing: first request locks the key, others wait for result. 2) Stale-while-revalidate: serve stale data immediately, refresh async in background. 3) Early refresh: background job refreshes keys before they expire. 4) Jittered TTL: add random offset (TTL + rand(0,60)) to prevent synchronized expiry.

## Q4: When would you use local (in-process) cache vs distributed cache (Redis)?
**Answer:** Local cache (Caffeine, Guava): ultra-low latency (<1μs), no network hop, no serialization. But: each instance has its own copy → inconsistency between pods, limited by pod memory, lost on restart. Use for: config, feature flags, hot reference data. Distributed cache (Redis): shared across instances → consistent view, survives pod restarts. But: network hop (~0.5ms), serialization cost. Use for: session data, user profiles, API responses. Best practice: two-level cache — local L1 (short TTL) → Redis L2 → DB.

## Q5: How do you monitor and optimize cache performance?
**Answer:** Key metrics: 1) Hit ratio (target >95%): `keyspace_hits / (hits + misses)`. Low ratio → wrong TTL or caching wrong data. 2) Evictions: should be ~0. If high → need more memory. 3) Memory usage: maxmemory + eviction policy (allkeys-lru). 4) P99 latency: should be <1ms. Spikes → check slow log (`SLOWLOG GET`). 5) Connection count: monitor for leaks. DevOps: set up Prometheus redis_exporter, alert on hit_ratio <80%, evictions >0, and memory >80%.

## Rapid-Fire
- **Cache-aside vs read-through?** → Cache-aside: app manages cache. Read-through: cache manages DB reads.
- **LRU vs LFU?** → LRU: evicts oldest unused. LFU: evicts least accessed. LFU better for skewed workloads.
- **Redis persistence modes?** → RDB: periodic snapshots. AOF: append every write. Both: RDB + AOF.
- **Cache warming?** → Pre-populate cache before traffic hits (deploy script, data pipeline).
- **Redis Cluster slots?** → 16,384 hash slots distributed across masters.
- **Bloom filter?** → Probabilistic: "definitely not in set" or "maybe in set". Prevents cache penetration.