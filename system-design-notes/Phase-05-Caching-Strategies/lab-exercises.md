# Phase 5: Caching Strategies — Lab Exercises

## Lab 1: Redis Cache-Aside Pattern
```bash
# Deploy Redis
kubectl create deployment redis --image=redis:7-alpine
kubectl expose deployment redis --port=6379

# Simulate cache-aside
kubectl exec -it deploy/redis -- redis-cli

# Write (after DB write):
DEL user:123                              # Invalidate
# Read:
GET user:123                              # Miss → null
SET user:123 '{"name":"alice"}' EX 300    # Cache from DB
GET user:123                              # Hit

# Monitor hit ratio:
INFO stats
# keyspace_hits: 5000
# keyspace_misses: 200
# Hit ratio = 5000 / 5200 = 96.1%
```

## Lab 2: Thundering Herd Simulation
```bash
# Simulate with parallel requests
# 1. Set a key
kubectl exec deploy/redis -- redis-cli SET hotkey "data" EX 5

# 2. Wait for expiry, then flood
for i in $(seq 1 100); do
  kubectl exec deploy/redis -- redis-cli GET hotkey &
done
wait
# All 100 return nil → all would hit DB

# Solution: Singleflight with SETNX lock
# Pseudocode:
# if GET hotkey == nil:
#   if SETNX hotkey:lock 1 EX 5:
#     data = query_db()
#     SET hotkey data EX 300
#     DEL hotkey:lock
#   else:
#     wait and retry GET hotkey
```

## Lab 3: Cache Eviction Policy Test
```bash
# Configure maxmemory and eviction
kubectl exec -it deploy/redis -- redis-cli
CONFIG SET maxmemory 10mb
CONFIG SET maxmemory-policy allkeys-lru

# Fill cache beyond limit
for i in $(seq 1 100000); do
  SET key:$i "value_padding_$(head -c 100 /dev/urandom | base64)" EX 600
done

# Check evictions:
INFO stats | grep evicted_keys
# evicted_keys should be > 0 once memory exceeded

# Compare policies:
#   allkeys-lru:     Evict any LRU key (best general)
#   volatile-lru:    Only evict keys with TTL
#   allkeys-lfu:     Evict least frequently used
#   noeviction:      Return error when full
```

## Lab 4: Cache Design Exercise
```
Scenario: E-commerce product catalog
  Products: 1M items, 500 bytes each
  Read: 50K QPS, Write: 100 QPS (admin updates)

  Design:
  1. CDN: Product images (Cache-Control: 1 year, versioned URLs)
  2. Redis: Product details (cache-aside, TTL=5 min)
     Memory: 1M × 500B = 500 MB (fits single Redis)
     Hit ratio target: 99% (products rarely change)
  3. Local cache: Top 1000 products (Caffeine, TTL=30s)
  4. Invalidation: Admin update → publish event → delete Redis key
  5. Thundering herd: Singleflight on popular products
  6. Cache warming: Pre-load top 10K products on deploy
```