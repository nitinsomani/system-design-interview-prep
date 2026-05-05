# Phase 1: System Design Fundamentals — Lab Exercises

## Lab 1: Back-of-Envelope Estimation Practice
```
Exercise: Estimate for a "Pastebin" service
  - 5M pastes/day, read:write = 5:1
  - Average paste size: 10 KB
  - Retention: 10 years

Calculate:
  1. Write QPS: 5M / 86400 ≈ 58 writes/sec, Peak ≈ 175/sec
  2. Read QPS: 58 × 5 = 290/sec, Peak ≈ 870/sec
  3. Storage/day: 5M × 10KB = 50 GB/day
  4. Storage/year: 50 × 365 = 18.25 TB/year
  5. 10 years: 182.5 TB
  6. Bandwidth write: 58 × 10KB = 580 KB/s
  7. Bandwidth read: 290 × 10KB = 2.9 MB/s
  8. Cache (20% hot): 5M × 0.2 × 10KB = 10 GB/day

Decision: S3 for storage, Redis 16GB for cache, 3 API servers
```

## Lab 2: Availability Calculation
```
Exercise: Calculate availability of this architecture:
  Web (99.99%) → API (99.95%) → DB Primary (99.99%) + Replica (99.99%)

  API path (series): 99.99% × 99.95% × DB = ?
  DB (parallel): 1 - (1-0.9999)(1-0.9999) = 1 - 0.00000001 = 99.999999%
  Total: 0.9999 × 0.9995 × 0.99999999 = 99.94%
  
  Bottleneck: The API server at 99.95% limits overall availability
  Fix: Add redundant API servers (parallel) → improves to 99.9999%
```

## Lab 3: Identify SPOFs
```
Exercise: Find all single points of failure:
  Users → DNS → LB → 3 API servers → 1 DB → 1 Redis

  SPOFs:
    1. DNS (single provider) → Add secondary DNS
    2. LB (single instance) → Cloud-managed or active-passive pair
    3. DB (single instance) → Add replica with auto-failover
    4. Redis (single instance) → Redis Sentinel or Cluster

  After fixing: Every component has redundancy
```

## Lab 4: Consistency Model Selection
```
Exercise: Choose consistency model for each:
  1. Bank account balance → Strong consistency (ACID)
  2. Instagram likes count → Eventual consistency
  3. User's own profile edit → Read-your-writes
  4. Chat message ordering → Causal consistency
  5. Product inventory (< 10 items) → Strong consistency
  6. News feed ranking → Eventual consistency
  7. Shopping cart → Eventual consistency (last-write-wins)
  8. Distributed lock → Strong consistency (CP system like etcd)
```