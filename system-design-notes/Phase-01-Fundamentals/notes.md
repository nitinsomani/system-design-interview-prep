# Phase 1: System Design Fundamentals — Notes

## Table of Contents
1. [What is System Design for DevOps?](#1-what-is-system-design-for-devops)
2. [Key Properties of Distributed Systems](#2-key-properties)
3. [CAP Theorem](#3-cap-theorem)
4. [Consistency Models](#4-consistency-models)
5. [Availability & Reliability](#5-availability-and-reliability)
6. [Latency & Throughput](#6-latency-and-throughput)
7. [Back-of-Envelope Estimation](#7-back-of-envelope-estimation)
8. [System Design Interview Framework](#8-system-design-interview-framework)

---

## 1. What is System Design for DevOps?

As a DevOps/SRE engineer, system design interviews focus on **how you build, deploy, scale, and operate** systems — not just the architecture on a whiteboard.

```
Software Engineer System Design:
  "Design Twitter" → data model, API, feed algorithm

DevOps/SRE System Design:
  "Design Twitter" → 
    How do you deploy it? (K8s, multi-region)
    How do you scale it? (auto-scaling, sharding)
    How do you monitor it? (SLIs, SLOs, alerting)
    How do you handle failures? (failover, circuit breaker)
    How do you deploy changes safely? (canary, rollback)
    What's the DR strategy? (RPO, RTO)
```

### DevOps Perspective on System Design

```
┌──────────────────────────────────────────────────┐
│           System Design for DevOps               │
├──────────────────────────────────────────────────┤
│                                                  │
│  Architecture     → Components, data flow        │
│  Infrastructure   → Cloud, K8s, networking       │
│  Deployment       → CI/CD, blue-green, canary    │
│  Scalability      → Auto-scaling, sharding       │
│  Reliability      → HA, failover, DR             │
│  Observability    → Metrics, logs, traces         │
│  Security         → IAM, secrets, encryption     │
│  Cost             → Right-sizing, reserved, spot │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 2. Key Properties of Distributed Systems

### Scalability

```
Vertical Scaling (Scale Up):
  Add more CPU, RAM, storage to existing machine
  Pros: Simple, no code changes
  Cons: Hardware limit, single point of failure, expensive
  Example: Upgrade RDS from db.r5.large → db.r5.4xlarge

Horizontal Scaling (Scale Out):
  Add more machines
  Pros: Near-infinite scale, fault tolerant, cost-effective
  Cons: Complexity (state, consistency, coordination)
  Example: Add more pods via HPA, more nodes via Cluster Autoscaler

Key insight for DevOps:
  Stateless services → easy to scale horizontally
  Stateful services → need careful planning (sharding, replication)
  
  Make everything stateless:
    Session → Redis/Memcached (shared session store)
    Files → S3/GCS (object storage)
    Config → ConfigMaps/Vault (external config)
```

### Availability

```
Availability = Uptime / (Uptime + Downtime)

Nines    Downtime/year    Downtime/month   Downtime/week
99%      3.65 days        7.3 hours        1.68 hours
99.9%    8.76 hours       43.8 minutes     10.1 minutes
99.99%   52.6 minutes     4.38 minutes     1.01 minutes
99.999%  5.26 minutes     26.3 seconds     6.05 seconds

Series availability (both must work):
  A(total) = A(1) × A(2)
  99.9% × 99.9% = 99.8%  (worse!)

Parallel availability (either works):
  A(total) = 1 - (1 - A(1)) × (1 - A(2))
  1 - (0.001 × 0.001) = 99.9999%  (better!)

Key insight: Redundancy improves availability
  Single DB (99.9%) vs Primary + Replica (99.9999%)
```

### Reliability

```
Reliability = system performs correctly under stated conditions

Reliability ≠ Availability:
  Available but unreliable: Server is up but returns wrong data
  Reliable but unavailable: During planned maintenance
  
Metrics:
  MTBF (Mean Time Between Failures): How often failures occur
  MTTR (Mean Time To Recovery): How fast you recover
  
  Availability = MTBF / (MTBF + MTTR)
  
  To improve availability:
    Increase MTBF → better hardware, testing, chaos engineering
    Decrease MTTR → automation, runbooks, monitoring
```

---

## 3. CAP Theorem

```
In a distributed system with network partitions, you can only guarantee
TWO of three properties:

  C — Consistency:  Every read gets the most recent write
  A — Availability: Every request gets a response (may not be latest)
  P — Partition Tolerance: System works despite network partitions

Since network partitions WILL happen (P is mandatory):
  You actually choose between C and A during a partition.

CP Systems (Consistency + Partition Tolerance):
  During partition: refuse requests rather than return stale data
  Examples: ZooKeeper, etcd, HBase, MongoDB (default)
  Use when: Financial transactions, inventory counts, leader election

AP Systems (Availability + Partition Tolerance):
  During partition: serve requests with possibly stale data
  Examples: Cassandra, DynamoDB, CouchDB, DNS
  Use when: Social media feeds, product catalog, shopping cart

Real-world nuance:
  Most systems aren't purely CP or AP — they're tunable.
  Cassandra: quorum reads = more consistent, ONE read = more available
  DynamoDB: strongly consistent read vs eventually consistent read
```

### PACELC Extension

```
When there's a Partition: choose Availability vs Consistency
Else (normal operation): choose Latency vs Consistency

P → A or C
E → L or C

Examples:
  DynamoDB: PA/EL  (available during partition, low latency normally)
  MongoDB:  PC/EC  (consistent during partition, consistent normally)
  Cassandra: PA/EL (available during partition, low latency normally)
```

---

## 4. Consistency Models

```
Strong Consistency:
  After a write, ALL subsequent reads return the new value
  Implementation: Single leader, synchronous replication
  Tradeoff: Higher latency, lower availability
  Example: Bank balance (must be accurate)

Eventual Consistency:
  After a write, reads MAY return old value temporarily
  Eventually (milliseconds to seconds), all reads return new value
  Implementation: Async replication, conflict resolution
  Tradeoff: Lower latency, higher availability
  Example: Social media likes count (slightly stale is OK)

Causal Consistency:
  If event A causes event B, everyone sees A before B
  Unrelated events may be seen in different order
  Example: Comment appears after the post it replies to

Read-Your-Writes:
  A user always sees their own writes immediately
  Other users may see stale data
  Example: After updating profile, you see the change
  Implementation: Read from leader for own data, replica for others

Monotonic Reads:
  Once you read a value, you never see an older value
  Example: After seeing 100 likes, you won't see 99
  Implementation: Sticky sessions to same replica
```

### Consistency in Practice (DevOps Relevance)

```
Database replication lag:
  Primary → Replica: 10-100ms typical
  Cross-region: 50-200ms
  
  Problem: User writes to primary (US-East), reads from replica (EU-West)
           Gets stale data for up to 200ms
  
  Solutions:
    1. Read from primary for critical reads (costly)
    2. Read-your-writes via session routing
    3. Accept eventual consistency for non-critical data
    4. Use global database (Aurora Global, CockroachDB)

K8s ConfigMap/Secret propagation:
  Update ConfigMap → takes ~1 minute to propagate to all pods
  This IS eventual consistency in your infrastructure!
  Solution: Rollout restart or use Reloader/Stakater
```

---

## 5. Availability and Reliability

### Achieving High Availability

```
Level 1: Single server (no HA)
  App → Single DB
  Availability: ~99% (one failure = total downtime)

Level 2: Redundancy within AZ
  App (2 pods) → DB Primary + Standby (same AZ)
  Availability: ~99.9%

Level 3: Multi-AZ
  App (pods across 3 AZs) → DB Multi-AZ (auto-failover)
  Availability: ~99.99%

Level 4: Multi-Region
  App (pods in US + EU) → DB (Global, cross-region replication)
  Availability: ~99.999%
  
Each level adds: complexity, cost, operational burden
Choose based on business requirement, not vanity
```

### Failure Modes

```
Single Point of Failure (SPOF):
  Any component whose failure takes down the system
  
  Common SPOFs:
    ✗ Single database instance
    ✗ Single load balancer
    ✗ Single DNS provider
    ✗ Single region deployment
    ✗ Single person who knows the system ("bus factor = 1")
  
  Eliminating SPOFs:
    ✓ Database: Primary + replica with auto-failover
    ✓ Load balancer: Redundant pair or cloud-managed
    ✓ DNS: Multiple providers (Route 53 + Cloudflare)
    ✓ Region: Multi-region active-active or active-passive
    ✓ Knowledge: Runbooks, documentation, cross-training
```

---

## 6. Latency and Throughput

### Latency Numbers Every DevOps Engineer Should Know

```
Operation                          Time
─────────────────────────────────────────
L1 cache reference                 0.5 ns
L2 cache reference                 7 ns
Main memory reference              100 ns
SSD random read                    150 μs
HDD random read (seek)             10 ms
Send 1 KB over 1 Gbps network     10 μs
Read 1 MB from SSD                 1 ms
Read 1 MB from HDD                 20 ms
Read 1 MB from network             10 ms
Roundtrip same datacenter          0.5 ms
Roundtrip US East ↔ West           40 ms
Roundtrip US ↔ Europe              80 ms
Roundtrip US ↔ Asia                150 ms

Key insight:
  Memory is 100,000x faster than disk
  SSD is 100x faster than HDD
  Same-DC network is 100x faster than cross-continent
  → Cache aggressively, use SSDs, stay in same region
```

### Throughput

```
Throughput = amount of work done per unit time

Web server:       10K-50K requests/second (single instance)
Database:         5K-20K queries/second (depends on query complexity)
Redis:            100K+ operations/second
Kafka:            1M+ messages/second (per broker)
Network (10Gbps): 1.25 GB/s

Throughput vs Latency tradeoff:
  Batch processing: high throughput, high latency
  Real-time processing: low latency, lower throughput
  
  Example: 
    Process 1 order at a time: 1ms latency, 1000 QPS
    Batch 100 orders: 100ms latency, 50000 QPS
```

---

## 7. Back-of-Envelope Estimation

### Example: Design a URL Shortener

```
Step 1: Traffic estimation
  100M URLs shortened per month
  Read:Write = 100:1
  Write QPS: 100M / (30 × 86400) ≈ 40 writes/sec
  Read QPS: 40 × 100 = 4000 reads/sec
  Peak: 2-3x → ~12000 reads/sec at peak

Step 2: Storage estimation
  Each URL record: ~500 bytes (short URL + long URL + metadata)
  Monthly: 100M × 500 bytes = 50 GB/month
  5 years: 50 GB × 60 = 3 TB total storage

Step 3: Bandwidth estimation
  Write: 40 × 500 bytes = 20 KB/s incoming
  Read: 4000 × 500 bytes = 2 MB/s outgoing

Step 4: Cache estimation
  80/20 rule: 20% of URLs generate 80% of traffic
  Cache 20% of daily reads:
  Daily reads: 4000 × 86400 = 345M
  Cache: 345M × 0.2 × 500 bytes ≈ 35 GB cache
  → Redis cluster with 40 GB memory

Decision:
  Storage: PostgreSQL (small data, ACID for writes)
  Cache: Redis (hot URLs)
  Servers: 3-5 instances behind LB (peak 12K QPS)
  CDN: For 301 redirects (reduce server load)
```

---

## 8. System Design Interview Framework

### The RADIO Framework (for DevOps)

```
R — Requirements:     Functional + non-functional + constraints
A — Architecture:     High-level components + data flow
D — Deep Dive:        Scaling, database, caching, messaging
I — Infrastructure:   Deployment, K8s, cloud, IaC
O — Operational:      Monitoring, alerting, DR, security, cost
```

### Template Answer Structure

```
"Let me start by clarifying the requirements..."
  - What's the expected scale? (users, QPS, data)
  - What's the availability target? (SLA)
  - Any compliance/regulatory requirements?

"Here's the high-level architecture..."
  [Draw components on whiteboard]
  - Client → CDN → LB → API → Cache → DB
  - Async: API → Queue → Workers

"Let me deep dive into [critical component]..."
  - Database choice and why
  - Caching strategy
  - Scaling approach

"From an infrastructure perspective..."
  - Deployed on K8s (EKS/GKE)
  - Multi-AZ, auto-scaling
  - IaC with Terraform
  - CI/CD with GitOps (ArgoCD)

"For operations and reliability..."
  - SLIs: latency P99, error rate, throughput
  - Alerting: PagerDuty for critical, Slack for warnings
  - DR: RPO < 1 hour, RTO < 15 minutes
  - Cost: ~$X/month estimation
```

---

> **Next**: Phase 2 covers High Availability & Reliability in depth.