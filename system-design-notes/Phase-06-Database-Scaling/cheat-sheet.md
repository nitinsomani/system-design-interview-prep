# Phase 6: Database Scaling — Cheat Sheet

## SQL vs NoSQL
```
SQL:           ACID, joins, schema → PostgreSQL, MySQL
Document:      Flexible schema, JSON → MongoDB, DynamoDB
Key-Value:     Fast lookups → Redis, DynamoDB
Wide-Column:   Write-heavy, time-series → Cassandra
Graph:         Relationships → Neo4j
```

## Replication
```
Sync:      Master waits for ACK → consistent, slower
Async:     Master doesn't wait → fast, eventual
Semi-sync: Wait for 1 replica → balanced
Read-after-write: Read from master for X sec after write
```

## Sharding Strategies
```
Range:       Simple but hot spots (date ranges)
Hash:        Even distribution, hard to range query
Directory:   Flexible, extra lookup overhead
Geographic:  Data locality + compliance
```

## Shard Key Rules
```
Good: user_id (even, query-aligned)
Bad:  timestamp (hot shard), country (skewed)
Rule: High cardinality + even distribution + query-aligned
```

## Connection Pooling
```
Problem: pods × conn_per_pod >> DB max_connections
PgBouncer: transaction mode, pool_size = max_conn / num_dbs
RDS Proxy: managed, IAM auth, failover-aware
```

## Index Types
```
B-tree:  Default, equality + range
Hash:    Equality only, fast
GIN:     Full-text, JSONB
Partial: WHERE clause filter
```

## Patterns
```
CQRS:     Write(SQL) → Event → Read(ES/Redis)
CDC:      Debezium → Kafka → sync downstream
Keyset:   WHERE id > last_id LIMIT 20 (not OFFSET)
```