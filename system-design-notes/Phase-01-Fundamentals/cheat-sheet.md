# Phase 1: System Design Fundamentals — Cheat Sheet

## CAP Theorem
```
C (Consistency) + A (Availability) + P (Partition Tolerance)
Network partitions are unavoidable → choose C or A during partition

CP: etcd, ZooKeeper, MongoDB → banking, inventory
AP: Cassandra, DynamoDB, DNS → social feeds, catalogs
```

## Consistency Models
```
Strong:           Every read = latest write (bank balance)
Eventual:         Reads may be stale temporarily (social likes)
Read-Your-Writes: You see your own changes immediately
Causal:           Cause-effect order preserved
```

## Availability Nines
```
99%    = 3.65 days/yr     99.9%  = 8.76 hrs/yr
99.99% = 52.6 min/yr      99.999% = 5.26 min/yr

Series:   A = A1 × A2           (both must work)
Parallel: A = 1-(1-A1)(1-A2)   (either works)
```

## Scaling
```
Vertical: Bigger machine (simple, limited, expensive)
Horizontal: More machines (complex, unlimited, cost-effective)
Rule: Make services stateless → easy horizontal scaling
```

## Latency Reference
```
L1 cache: 0.5ns    RAM: 100ns     SSD: 150μs    HDD: 10ms
Same DC: 0.5ms     US coast: 40ms  US↔EU: 80ms   US↔Asia: 150ms
```

## Quick Estimation
```
QPS = DAU × queries/user / 86400    Peak = 2-3x average
Storage = QPS × size × 86400       Cache = 20% of daily reads
Servers = Peak QPS / QPS_per_server (10K-50K per instance)
```

## ACID vs BASE
```
ACID: Atomicity, Consistency, Isolation, Durability (SQL)
BASE: Basically Available, Soft state, Eventually consistent (NoSQL)
```

## Interview Framework (RADIO)
```
R: Requirements (functional + non-functional)
A: Architecture (components + data flow)
D: Deep Dive (DB, cache, scaling)
I: Infrastructure (K8s, cloud, IaC)
O: Operational (monitoring, DR, security, cost)
```