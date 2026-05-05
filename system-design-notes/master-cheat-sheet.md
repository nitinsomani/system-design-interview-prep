# System Design for DevOps Engineers — Master Cheat Sheet & Roadmap

> **Goal**: Complete system design preparation for Product-Based Company DevOps/SRE roles
> **Structure**: Each phase has 4 files — `notes.md`, `cheat-sheet.md`, `interview-qa.md`, `lab-exercises.md`
> **Total**: 15 phases × 4 files = **60 files**

---

## Progress Tracker

| # | Phase | Status | Key Topics |
|---|-------|--------|-----------|
| 01 | **System Design Fundamentals** | ✅ | CAP, consistency, availability, latency, throughput, SLI/SLO |
| 02 | **High Availability & Reliability** | ✅ | Redundancy, failover, replication, DR, chaos engineering |
| 03 | **Scalability Patterns** | ✅ | Horizontal/vertical, auto-scaling, sharding, stateless design |
| 04 | **Load Balancing & Traffic Mgmt** | ✅ | L4/L7 LB, global LB, CDN, traffic shaping, rate limiting |
| 05 | **Caching Strategies** | ✅ | Redis, Memcached, CDN, cache patterns, invalidation |
| 06 | **Database Design & Scaling** | ✅ | SQL/NoSQL, replication, sharding, connection pooling |
| 07 | **Message Queues & Async** | ✅ | Kafka, RabbitMQ, SQS, event sourcing, CQRS |
| 08 | **Storage & Data Management** | ✅ | Object/block/file storage, data lifecycle, backup |
| 09 | **Monitoring & SRE Practices** | ✅ | SLI/SLO/SLA, error budgets, alerting, incident mgmt |
| 10 | **CI/CD & Deployment Strategies** | ✅ | Blue-green, canary, GitOps, feature flags, rollback |
| 11 | **Containers & Orchestration** | ✅ | Docker, K8s, Helm, operators, multi-tenancy |
| 12 | **Infrastructure as Code** | ✅ | Terraform, Ansible, Pulumi, state mgmt, drift |
| 13 | **Security Architecture** | ✅ | IAM, secrets, zero trust, compliance, supply chain |
| 14 | **Cloud Architecture Patterns** | ✅ | Multi-cloud, hybrid, serverless, cost optimization |
| 15 | **Interview Preparation** | ✅ | Design exercises, frameworks, mock scenarios |

---

## System Design Framework for DevOps

```
1. CLARIFY Requirements (2-3 min)
   - Functional: What does the system do?
   - Non-functional: Scale, latency, availability, consistency?
   - Constraints: Budget, team size, compliance?

2. ESTIMATE Scale (2-3 min)
   - Users: DAU, peak concurrent
   - Traffic: Requests/second, read:write ratio
   - Storage: Data size, growth rate, retention
   - Bandwidth: Ingress/egress per day

3. HIGH-LEVEL Design (5-7 min)
   - Draw major components
   - Define APIs / data flow
   - Choose communication patterns (sync/async)

4. DEEP DIVE Components (10-15 min)
   - Database choice + schema
   - Caching strategy
   - Scaling approach
   - Deployment architecture

5. OPERATIONAL Concerns (5 min)
   - Monitoring & alerting
   - Deployment strategy
   - Disaster recovery
   - Security
   - Cost optimization
```

## Quick Reference: Key Numbers

```
Latency:
  L1 cache:           0.5 ns
  L2 cache:           7 ns
  RAM access:         100 ns
  SSD read:           150 μs
  HDD seek:           10 ms
  Same datacenter RT: 0.5 ms
  US East↔West:       40 ms
  US↔Europe:          75 ms
  US↔Asia:            150 ms

Throughput:
  SSD sequential:     500 MB/s (SATA), 3 GB/s (NVMe)
  Network (10 Gbps):  1.25 GB/s
  HDD sequential:     100 MB/s

Scale:
  1 million = 10^6    1 billion = 10^9
  86,400 seconds/day  2.6M seconds/month
  1 KB = 1,000 bytes  1 MB = 10^6  1 GB = 10^9  1 TB = 10^12

Availability:
  99.9%  = 8.76 hr/yr downtime   (three nines)
  99.99% = 52.6 min/yr downtime  (four nines)
  99.999% = 5.26 min/yr downtime (five nines)
```

## Essential Formulas

```
QPS (Queries Per Second):
  DAU × avg_queries_per_user / 86400
  Peak QPS ≈ 2-3x average QPS

Storage:
  Daily: QPS × data_size_per_query × 86400
  Yearly: Daily × 365
  5-year: Yearly × 5

Bandwidth:
  Ingress: QPS × request_size
  Egress:  QPS × response_size

Servers needed:
  Total QPS / QPS_per_server
  (single server ≈ 10K-50K QPS for simple services)
```

---

## Core Concepts Quick Reference

```
CAP Theorem:      Choose 2 of 3: Consistency, Availability, Partition tolerance
                  CP: Strong consistency (banking) → sacrifice availability
                  AP: High availability (social media) → eventual consistency

ACID:             Atomicity, Consistency, Isolation, Durability (SQL)
BASE:             Basically Available, Soft state, Eventually consistent (NoSQL)

Vertical Scale:   Bigger machine (limited ceiling)
Horizontal Scale: More machines (preferred, needs stateless design)

Stateless:        No server-side session → any instance handles any request
Stateful:         Session tied to server → needs sticky sessions or shared state
```

## Database Decision Guide

```
Need ACID + relationships?     → PostgreSQL / MySQL
Need flexible schema?          → MongoDB / DynamoDB
Need time-series data?         → InfluxDB / TimescaleDB
Need full-text search?         → Elasticsearch / OpenSearch
Need graph relationships?      → Neo4j / Neptune
Need key-value at scale?       → Redis / DynamoDB
Need wide-column analytics?    → Cassandra / ScyllaDB
Need data warehouse?           → BigQuery / Redshift / Snowflake
```

## Caching Decision Guide

```
Cache-Aside:     App checks cache → miss → read DB → write cache
Read-Through:    Cache auto-loads from DB on miss
Write-Through:   Write to cache + DB simultaneously
Write-Behind:    Write to cache → async flush to DB (risk: data loss)
Write-Around:    Write to DB only → cache populated on read miss

Where to cache:
  Browser:       Static assets, API responses (Cache-Control headers)
  CDN:           Static + dynamic content at edge
  API Gateway:   Response caching per endpoint
  Application:   In-memory (local) or distributed (Redis)
  Database:      Query cache, materialized views
```

## Message Queue Decision Guide

```
Need ordering + replay?          → Kafka
Need routing + flexibility?      → RabbitMQ
Need simple cloud-native queue?  → SQS / Cloud Pub/Sub
Need real-time pub/sub?          → Redis Pub/Sub / NATS
Need exactly-once processing?    → Kafka (with idempotent consumers)
```

---

## Study Strategy

1. **Read** `notes.md` for deep understanding
2. **Do** `lab-exercises.md` hands-on
3. **Review** `cheat-sheet.md` for quick revision
4. **Practice** `interview-qa.md` by explaining answers out loud
5. **Repeat** — spaced repetition is key

### Study Order (by interview frequency)
```
High Priority:  Phase 1, 3, 6, 5, 2       (fundamentals + DB + caching)
Medium:         Phase 7, 4, 9, 10          (queues + LB + SRE + CI/CD)
Good to Know:   Phase 11, 12, 13, 14, 8    (containers + IaC + security)
Final:          Phase 15                    (mock interviews)
```

---

> **All 15 phases complete. 60 files ready for review. Good luck!**