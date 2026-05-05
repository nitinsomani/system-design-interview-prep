# Phase 5: Caching Strategies — Notes

## 1. Why Caching Matters

```
Without cache:  DB query = 5-50ms per request
With cache:     Redis lookup = 0.5ms per request  (10-100x faster)

Cache hit ratio of 90% → 10x fewer DB queries
  1000 QPS → only 100 reach DB

Cache is the #1 tool for scaling reads
```

## 2. Caching Patterns

```
Cache-Aside (Lazy Loading):
  Read: Check cache → miss → read DB → write cache → return
  Write: Write DB → delete cache
  Pros: Only caches what's needed, resilient to cache failure
  Cons: First read always slow (cache miss), stale data possible
  Best for: Most use cases, read-heavy workloads

Read-Through:
  Cache sits in front of DB, handles reads automatically
  Read: App → Cache → (miss) → Cache reads DB → returns
  Pros: App logic simpler
  Cons: Cache library must support, harder to debug

Write-Through:
  Write: App → Cache → Cache writes DB (synchronous)
  Pros: Cache always consistent with DB
  Cons: Write latency increases, cache may have unread data

Write-Behind (Write-Back):
  Write: App → Cache → return (async write to DB later)
  Pros: Fastest writes, batch DB writes
  Cons: Data loss risk if cache crashes before DB write
  Use for: Analytics, counters, non-critical data

Write-Around:
  Write: App → DB directly (skip cache)
  Read: Cache-aside for reads
  Pros: No cache pollution from writes
  Cons: Read-after-write always misses cache
```

## 3. Cache Invalidation Strategies

```
TTL (Time to Live):
  Set expiry: SET key value EX 300  (5 minutes)
  Simple, predictable
  Stale for up to TTL duration

Event-Driven Invalidation:
  On DB write → publish event → delete cache key
  More consistent than TTL
  Implementation: CDC (Debezium) → Kafka → cache invalidator

Versioned Keys:
  user:123:v5 → bump version on write
  Old keys expire via TTL
  No explicit deletion needed

The two hardest problems in CS:
  1. Cache invalidation
  2. Naming things
  3. Off-by-one errors
```

## 4. Cache Problems & Solutions

```
Thundering Herd (Cache Stampede):
  Problem: Popular key expires → 1000 concurrent requests hit DB
  Solutions:
    1. Singleflight: First request fetches, others wait
    2. Stale-while-revalidate: Serve stale, async refresh
    3. Locking: Distributed lock on cache key during refresh
    4. Pre-warm: Refresh before expiry

Hot Key:
  Problem: One key gets disproportionate traffic (celebrity profile)
  Solutions:
    1. Local cache (in-process) for hottest keys
    2. Key replication: user:123:shard1, user:123:shard2
    3. Rate limit reads to origin

Cache Penetration:
  Problem: Requests for non-existent data bypass cache, always hit DB
  Solutions:
    1. Cache null results with short TTL
    2. Bloom filter: check membership before DB query

Cache Avalanche:
  Problem: Many keys expire at same time → mass DB load
  Solutions:
    1. Jittered TTL: TTL + random(0, 60s)
    2. Staggered cache warming
```

## 5. Redis vs Memcached

```
Feature          Redis              Memcached
Data types       Strings, Hash,     Strings only
                 List, Set, Sorted
                 Set, Streams
Persistence      RDB + AOF          None
Replication      Master-Replica     None (client-side)
Cluster          Redis Cluster      Client-side sharding
Pub/Sub          Yes                No
Lua scripting    Yes                No
Max value size   512 MB             1 MB
Threading        Single (6.0+ I/O   Multi-threaded
                 threads)

When to use Redis:    Complex data, persistence, pub/sub
When to use Memcached: Simple key-value, multi-threaded perf
```

## 6. Multi-Level Caching

```
Level 1: Browser Cache (Cache-Control headers)
Level 2: CDN Cache (CloudFront, edge locations)
Level 3: API Gateway Cache (short TTL, per-route)
Level 4: Application Cache (in-process, Caffeine/Guava)
Level 5: Distributed Cache (Redis/Memcached)
Level 6: Database Cache (query cache, buffer pool)

Request flow:
  Browser → CDN → API GW → App Local → Redis → DB
  Each level reduces load on the next
```

---

> **DevOps focus**: Redis cluster operations, cache monitoring (hit ratio, eviction rate), TTL strategy, thundering herd prevention.