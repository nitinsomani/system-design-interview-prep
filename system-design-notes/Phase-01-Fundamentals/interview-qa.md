# Phase 1: System Design Fundamentals — Interview Q&A

## Q1: What is the CAP theorem and how does it apply in practice?
**Answer:** CAP states that during a network partition, you choose consistency or availability. CP systems (etcd, ZooKeeper) refuse requests rather than return stale data — used for leader election, config management. AP systems (Cassandra, DynamoDB) serve possibly stale data to stay available — used for social feeds, caches. In practice, most systems are tunable: DynamoDB offers both strongly consistent and eventually consistent reads.

## Q2: Explain the difference between availability and reliability.
**Answer:** Availability = system is reachable and responds. Reliability = system responds correctly. A system can be available but unreliable (returns wrong data), or reliable but unavailable (during maintenance). We measure availability in nines (99.9% = 8.76 hrs downtime/year), and reliability via MTBF/MTTR. As DevOps, I improve availability by adding redundancy (parallel systems) and decrease MTTR through automation and runbooks.

## Q3: How do you estimate the scale of a system?
**Answer:** I use back-of-envelope math: DAU × queries/user ÷ 86400 = avg QPS. Peak = 2-3x average. Storage = QPS × record size × 86400 × retention days. Bandwidth = QPS × payload size. Cache = 20% of daily data (80/20 rule). This helps choose database size, number of servers, and cache capacity before deep-diving into architecture.

## Q4: Vertical vs horizontal scaling — when do you use each?
**Answer:** Vertical (bigger machine) is simpler but has a ceiling and creates SPOF. Horizontal (more machines) scales indefinitely but requires stateless design. For DevOps: databases often scale vertically first (RDS instance upgrade), then horizontally (read replicas, sharding). Stateless services scale horizontally from day 1 via K8s HPA. I always push for stateless design — externalize sessions to Redis, files to S3, config to Vault.

## Q5: What are SLIs, SLOs, and SLAs?
**Answer:** SLI (Indicator) = metric you measure (P99 latency, error rate). SLO (Objective) = target for that metric (P99 < 200ms, 99.9% availability). SLA (Agreement) = contract with consequences (99.9% uptime or credit). As SRE, I define SLIs first, set SLOs based on user experience, and track error budget (100% - SLO). If error budget is healthy, we deploy freely; if depleted, we freeze changes and focus on reliability.

## Rapid-Fire
- **ACID?** → Atomicity, Consistency, Isolation, Durability (SQL guarantees)
- **BASE?** → Basically Available, Soft state, Eventual consistency (NoSQL)
- **MTBF?** → Mean Time Between Failures (how often things break)
- **MTTR?** → Mean Time To Recovery (how fast you fix it)
- **Stateless service?** → No server-side session; any instance handles any request
- **SPOF?** → Single Point of Failure; eliminate with redundancy
- **RPO vs RTO?** → RPO: max data loss tolerable. RTO: max downtime tolerable
- **Throughput vs latency?** → Throughput: volume/time. Latency: time per request
- **Read-heavy system?** → Add read replicas + caching (Redis)
- **Write-heavy system?** → Sharding + async processing (message queue)