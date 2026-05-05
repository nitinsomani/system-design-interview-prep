# Phase 5: Caching Strategies — Cheat Sheet

## Caching Patterns
```
Cache-Aside:    Read miss → DB → cache; Write → DB → delete cache
Read-Through:   Cache reads DB on miss automatically
Write-Through:  Cache writes DB synchronously
Write-Behind:   Cache writes DB asynchronously (batch)
Write-Around:   Write DB directly, skip cache
```

## Eviction Policies
```
LRU:  Least Recently Used (default, best general purpose)
LFU:  Least Frequently Used (good for skewed access)
TTL:  Time-based expiry (predictable freshness)
FIFO: First In First Out (simple)
```

## Cache Problems
```
Thundering Herd:   Hot key expires → singleflight/lock/pre-warm
Hot Key:           One key overloaded → local cache + key replication
Cache Penetration: Non-existent keys → cache null + bloom filter
Cache Avalanche:   Mass expiry → jittered TTL
```

## Redis Quick Reference
```bash
SET key val EX 300          # Set with 5 min TTL
GET key                     # Read
DEL key                     # Delete (invalidation)
MGET k1 k2 k3              # Batch read
INFO stats                  # Hit ratio: keyspace_hits/misses
redis-cli --latency         # Measure latency
```

## Key Metrics
```
Hit Ratio:     > 95% good, < 80% investigate
Evictions:     Should be ~0 (increase memory if high)
Memory Usage:  Monitor with INFO memory
Latency:       P99 < 1ms typical
Connections:   Monitor with INFO clients
```

## Redis vs Memcached
```
Redis:     Rich types, persistence, pub/sub, Lua, cluster
Memcached: Simple strings, multi-threaded, no persistence
Default choice: Redis (unless you specifically need multi-thread perf)
```