# Phase 8: Storage & Data Management — Cheat Sheet

## Storage Types
```
Block (EBS):   Single attach, <1ms, DB workloads
File (EFS):    Shared, 1-10ms, POSIX, CMS/legacy
Object (S3):   Unlimited, 10-100ms, backups/logs/data lake
```

## S3 Storage Classes
```
Standard:      Frequent access           $0.023/GB
Standard-IA:   Infrequent, 30d min       $0.0125/GB
Glacier Inst:  Archive, ms retrieval     $0.004/GB
Glacier Deep:  Long-term, 12-48hr        $0.00099/GB
```

## Lifecycle Rules
```
Standard → IA (30 days) → Glacier (90 days) → Delete (365 days)
Saves 60-80% storage costs
```

## Backup Strategy (3-2-1)
```
3 copies, 2 media types, 1 offsite
Full → Incremental → Differential
RDS: Automated snapshots + point-in-time recovery (5 min)
K8s: Velero (resources + PV snapshots)
```

## Data Retention
```
Logs:     30-90d hot → 1yr cold → delete
Metrics:  15d full → 1yr downsampled
Backups:  30d daily → 12m monthly → 7yr yearly
```

## Data Pipeline
```
ETL: Extract → Transform → Load (traditional)
ELT: Extract → Load → Transform (modern, transform in warehouse)
CDC: Debezium → Kafka → downstream
Data Lake: S3 + Athena (schema-on-read)
Data Warehouse: Snowflake/BigQuery (columnar, analytics)
```