# Phase 6: Database Scaling — Notes

## 1. SQL vs NoSQL Decision

```
SQL (PostgreSQL, MySQL):
  Strong consistency (ACID)
  Complex queries, joins, aggregations
  Schema enforced, relational data
  Vertical scaling + read replicas
  Use when: Financial data, relationships matter, complex queries

NoSQL types:
  Document (MongoDB, DynamoDB):     JSON docs, flexible schema
  Key-Value (Redis, DynamoDB):      Simple lookups, ultra-fast
  Wide-Column (Cassandra, HBase):   Time-series, high write throughput
  Graph (Neo4j, Neptune):           Relationships, social networks

Decision matrix:
  Need transactions?           → SQL
  Schema changes frequently?   → Document NoSQL
  Need extreme write scale?    → Wide-column (Cassandra)
  Simple key lookups?          → Key-value
  Complex relationships?       → Graph DB
```

## 2. Replication

```
Master-Replica (Read Replicas):
  Master: handles all writes
  Replicas: handle reads (scale reads horizontally)
  Replication lag: async = ms to seconds

  Synchronous:   Master waits for replica ACK → strong consistency, slower
  Asynchronous:  Master doesn't wait → fast, eventual consistency
  Semi-sync:     Master waits for 1 replica → balance

  Read-after-write problem:
    User writes → reads from replica → data not there yet
    Solutions:
      1. Read from master for X seconds after write
      2. Use replica that's caught up (check LSN)
      3. Session-level consistency (sticky reads)
```

## 3. Sharding (Horizontal Partitioning)

```
Split data across multiple DB instances

Strategies:
  Range-based:      user_id 1-1M → shard1, 1M-2M → shard2
    Pros: Simple, range queries easy
    Cons: Hot spots (recent data on one shard)

  Hash-based:       shard = hash(user_id) % N
    Pros: Even distribution
    Cons: Range queries need scatter-gather, resharding is painful

  Directory-based:  Lookup table maps key → shard
    Pros: Flexible, can move keys
    Cons: Directory is SPOF, extra lookup

  Geographic:       Region-based (EU users → EU shard)
    Pros: Data locality, compliance (GDPR)
    Cons: Cross-region queries complex

Shard key selection (critical):
  Good: user_id (even distribution, queries are user-scoped)
  Bad:  created_date (all writes go to latest shard)
  Bad:  country (uneven — US shard 100x bigger than Iceland)

Cross-shard queries:
  JOIN across shards → scatter-gather (expensive)
  Solution: denormalize, or use separate analytics DB
```

## 4. Connection Management

```
Problem: 500 pods × 20 connections = 10,000 DB connections
  PostgreSQL max_connections: typically 200-500

Solution: Connection Pooling
  PgBouncer:  External pooler, sits between app and DB
    Transaction mode: conn returned after each transaction
    Session mode: conn held for session lifetime
    Statement mode: conn returned after each statement

  RDS Proxy:  AWS managed, IAM auth, failover-aware

Pool sizing formula (PgBouncer):
  default_pool_size = DB_max_connections / num_databases
  Example: 200 DB conns / 4 DBs = 50 per pool
```

## 5. Database Patterns

```
CQRS (Command Query Responsibility Segregation):
  Separate write model (normalized) from read model (denormalized)
  Write → PostgreSQL (transactions, constraints)
  Read → Elasticsearch/Redis (optimized for queries)
  Sync via events (CDC/Kafka)

Event Sourcing:
  Store events, not current state
  Account: [Deposit $100, Withdraw $30, Deposit $50]
  Current state: replay events → $120
  Pros: Full audit trail, temporal queries
  Cons: Complex, eventual consistency

Change Data Capture (CDC):
  Capture DB changes as events
  Tools: Debezium (Kafka Connect), AWS DMS
  Use: Sync to cache, search index, analytics DB
```

## 6. Indexing & Query Optimization

```
Index types:
  B-tree:    Default, good for equality + range (WHERE id = 5, date > X)
  Hash:      Equality only, faster than B-tree for exact match
  GIN:       Full-text search, JSONB, arrays
  GiST:      Geometric, geographic data

Rules:
  - Index columns in WHERE, JOIN, ORDER BY
  - Composite index: order matters (leftmost prefix)
  - Don't over-index: each index slows writes
  - EXPLAIN ANALYZE every slow query
  - Partial indexes: CREATE INDEX ON orders(status) WHERE status = 'pending'

Query optimization:
  N+1 problem: 1 query for list + N queries for details → use JOIN
  SELECT *: fetch only needed columns
  OFFSET pagination: slow for large offsets → use cursor/keyset pagination
```

---

> **DevOps focus**: Read replica setup, connection pooling (PgBouncer/RDS Proxy), CDC pipeline, backup/restore, monitoring slow queries.