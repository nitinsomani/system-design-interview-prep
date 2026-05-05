# Phase 6: Database Scaling — Interview Q&A

## Q1: SQL vs NoSQL — how do you decide?
**Answer:** Start with SQL (PostgreSQL) unless you have a specific reason not to. SQL gives you ACID transactions, complex queries, and a mature ecosystem. Switch to NoSQL when: 1) Schema changes frequently and you need flexibility → MongoDB. 2) Need extreme write throughput (>100K writes/sec) → Cassandra. 3) Simple key-value lookups at ultra-low latency → DynamoDB/Redis. 4) Heavy relationship traversal → Neo4j. Many systems use polyglot persistence: SQL for core transactions + Redis for cache + Elasticsearch for search.

## Q2: How would you scale a database handling 100K reads/sec?
**Answer:** Layered approach: 1) Caching: Redis cache with 95%+ hit ratio → only 5K QPS reach DB. 2) Read replicas: 3-5 replicas, each handles ~10K QPS. 3) Connection pooling: PgBouncer in transaction mode to multiplex connections. 4) Query optimization: proper indexes, EXPLAIN ANALYZE slow queries, eliminate N+1. 5) If still not enough: CQRS — denormalized read model in Elasticsearch. For writes: if >10K writes/sec on single table, consider sharding.

## Q3: Explain database sharding and its challenges.
**Answer:** Sharding splits data across multiple DB instances horizontally. Choose shard key carefully: user_id is common (even distribution, queries are user-scoped). Challenges: 1) Cross-shard queries/joins are expensive (scatter-gather). 2) Resharding when adding nodes is complex (consistent hashing helps). 3) Distributed transactions across shards need 2PC or saga pattern. 4) Auto-increment IDs don't work → use UUIDs or snowflake IDs. 5) Operational complexity: backup, migration, monitoring multiply by N. Alternative to manual sharding: Vitess (MySQL), Citus (PostgreSQL), or DynamoDB (auto-sharded).

## Q4: What is the read-after-write consistency problem?
**Answer:** User writes data to master, then immediately reads from a replica that hasn't received the replication yet → sees stale data. Solutions: 1) Read from master for X seconds after write (track last write timestamp). 2) Check replica LSN/position — only read from caught-up replicas. 3) Synchronous replication (guarantees but slower). 4) Session-level consistency: sticky the user's reads to a specific replica. In practice: most apps use async replication + read-from-master-after-write for critical paths only.

## Q5: How do you handle database migrations in production with zero downtime?
**Answer:** Expand-contract pattern: 1) **Expand**: Add new column/table (nullable, no constraints). Deploy code that writes to both old and new. 2) **Migrate**: Backfill existing data. 3) **Contract**: Deploy code that reads from new only. Add constraints. Drop old column. Tools: Flyway, Liquibase, or plain SQL migrations. Rules: never rename/drop columns in one step, never add NOT NULL without default, always test migrations against production-size data. For large tables: use pt-online-schema-change (MySQL) or pg_repack (PostgreSQL) to avoid locking.

## Rapid-Fire
- **ACID?** → Atomicity, Consistency, Isolation, Durability
- **BASE?** → Basically Available, Soft state, Eventually consistent
- **N+1 query?** → 1 query for list + N queries for each item → use JOIN or batch
- **Cursor vs offset pagination?** → Cursor: WHERE id > last_id (fast). Offset: skip N rows (slow for large N)
- **Connection pooling modes?** → Transaction: return conn after txn. Session: hold for session. Statement: per query.
- **WAL?** → Write-Ahead Log: write to log before data file, ensures crash recovery
- **Vacuum in PostgreSQL?** → Reclaims dead tuples from MVCC. Autovacuum should be tuned, not disabled.