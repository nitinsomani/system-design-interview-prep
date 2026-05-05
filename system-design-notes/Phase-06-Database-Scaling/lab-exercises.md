# Phase 6: Database Scaling — Lab Exercises

## Lab 1: PostgreSQL Read Replica Setup
```bash
# Primary (docker-compose concept)
# postgresql.conf:
#   wal_level = replica
#   max_wal_senders = 3

# Replica:
#   primary_conninfo = 'host=primary port=5432 user=replicator'
#   primary_slot_name = 'replica1'

# Check replication lag:
SELECT
  client_addr,
  state,
  pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes
FROM pg_stat_replication;

# Application routing:
# Writes → primary:5432
# Reads  → replica:5433
```

## Lab 2: Connection Pooling with PgBouncer
```bash
# pgbouncer.ini
# [databases]
# mydb = host=postgres port=5432 dbname=mydb
# [pgbouncer]
# pool_mode = transaction
# max_client_conn = 1000
# default_pool_size = 25
# reserve_pool_size = 5
# reserve_pool_timeout = 3

# Monitor pool:
# SHOW POOLS;   → active, waiting, server connections
# SHOW STATS;   → requests, bytes, query time

# Before PgBouncer: 500 pods × 10 conn = 5000 DB connections (crash)
# After PgBouncer:  500 pods → PgBouncer → 25 DB connections (fine)
```

## Lab 3: Sharding Design Exercise
```
Scenario: Social media with 100M users, 10B posts
  Requirement: Get user's posts, get user's feed

  Shard key analysis:
  Option A: user_id
    ✅ Get user's posts: single shard query
    ❌ Get feed (friends' posts): cross-shard (scatter-gather)
    
  Option B: post_id
    ❌ Get user's posts: cross-shard
    ❌ Get feed: cross-shard
    → Worse than user_id

  Decision: Shard by user_id
    User posts → single shard lookup
    Feed → fan-out-on-write (pre-compute feed per user)
    Feed table also sharded by user_id → single shard read

  Shard count:
    100M users × 1KB = 100 GB per shard type
    10B posts × 500B = 5 TB
    With 16 shards: ~312 GB/shard → manageable
```

## Lab 4: Index Optimization Practice
```sql
-- Slow query: find pending orders for a user
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 123 AND status = 'pending'
ORDER BY created_at DESC LIMIT 20;

-- Without index: Seq Scan (full table scan) → 2000ms

-- Add composite index:
CREATE INDEX idx_orders_user_status_date
ON orders(user_id, status, created_at DESC);
-- With index: Index Scan → 0.5ms

-- Partial index (if most queries are for pending):
CREATE INDEX idx_orders_pending
ON orders(user_id, created_at DESC)
WHERE status = 'pending';
-- Even faster: smaller index, only pending rows

-- Check index usage:
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
-- idx_scan = 0 → unused index, consider dropping
```