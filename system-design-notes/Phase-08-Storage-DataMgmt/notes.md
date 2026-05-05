# Phase 8: Storage & Data Management — Notes

## 1. Storage Types

```
Block Storage (EBS, Persistent Volumes):
  - Attached to single instance/pod
  - Low latency, high IOPS
  - Use for: databases, transactional workloads
  - Types: gp3 (general), io2 (high IOPS), st1 (throughput)

File Storage (EFS, NFS):
  - Shared across multiple instances/pods
  - POSIX-compatible filesystem
  - Use for: shared config, CMS, legacy apps
  - Higher latency than block

Object Storage (S3, GCS, MinIO):
  - Flat namespace, key-value (key = path, value = blob)
  - Unlimited scale, 99.999999999% durability (11 nines)
  - Use for: backups, logs, images, data lake, static hosting
  - Not suitable for: frequent updates, low-latency access

Comparison:
  Type     Latency    Scale         Shared    Cost
  Block    <1ms       TB            No        $$$
  File     1-10ms     PB            Yes       $$
  Object   10-100ms   Unlimited     Yes       $
```

## 2. S3 Deep Dive

```
Storage Classes:
  Standard:          Frequent access                $0.023/GB
  Intelligent:       Auto-tier based on access      $0.023/GB
  Standard-IA:       Infrequent, min 30 days        $0.0125/GB
  One Zone-IA:       Single AZ, infrequent          $0.01/GB
  Glacier Instant:   Archive, ms retrieval          $0.004/GB
  Glacier Flexible:  Archive, min-hours retrieval   $0.0036/GB
  Glacier Deep:      Long-term, 12-48hr retrieval   $0.00099/GB

Lifecycle Rules:
  - Standard → IA after 30 days
  - IA → Glacier after 90 days
  - Delete after 365 days
  Saves 60-80% on storage costs

Performance:
  - 5,500 GET/s per prefix
  - 3,500 PUT/s per prefix
  - Scale by distributing across prefixes
  - Use S3 Transfer Acceleration for cross-region uploads

Security:
  - Bucket policy (resource-based)
  - IAM policy (identity-based)
  - Block public access (account + bucket level)
  - Encryption: SSE-S3, SSE-KMS, SSE-C
  - Access logs + CloudTrail
```

## 3. Data Backup Strategies

```
3-2-1 Rule:
  3 copies of data
  2 different media types
  1 offsite/different region

Backup types:
  Full:          Complete copy (slow, large)
  Incremental:   Only changes since last backup (fast, small)
  Differential:  Changes since last full backup (medium)

Database backups:
  PostgreSQL:    pg_dump (logical), pg_basebackup (physical)
  MySQL:         mysqldump, xtrabackup (hot backup)
  MongoDB:       mongodump, filesystem snapshots
  
  RDS:           Automated snapshots (up to 35 days retention)
                 Manual snapshots (persist until deleted)
                 Point-in-time recovery (5-min granularity)

Kubernetes:
  Velero:        Backup K8s resources + PV snapshots
                 Schedule: daily backup, 30-day retention
                 Restore: full cluster or namespace-level
```

## 4. Data Lifecycle Management

```
Stages:
  Hot:    Active, frequently accessed (SSD, Redis, S3 Standard)
  Warm:   Less frequent (S3-IA, cheaper disks)
  Cold:   Archival, rare access (Glacier, tape)
  Delete: End of retention, compliance-safe deletion

  Data tiering automation:
    - S3 Lifecycle rules
    - Custom: track last_accessed → move to cheaper tier

Data retention:
  Logs:          30-90 days hot → 1 year cold → delete
  Metrics:       15 days full resolution → 1 year downsampled
  Backups:       30 days daily → 12 months monthly → 7 years yearly
  User data:     Per policy/regulation (GDPR: delete on request)
```

## 5. Data Pipeline Architecture

```
ETL vs ELT:
  ETL: Extract → Transform → Load (traditional, transform before loading)
  ELT: Extract → Load → Transform (modern, transform in data warehouse)

  Tools:
    Extract:    Debezium (CDC), Kafka Connect, Fivetran
    Transform:  dbt, Spark, custom
    Load:       Snowflake, BigQuery, Redshift

Data Lake:
  Raw data in S3 (any format: JSON, Parquet, CSV)
  Schema-on-read (not schema-on-write)
  Query with Athena, Presto, Spark

Data Warehouse:
  Structured, optimized for analytics
  Snowflake, BigQuery, Redshift
  Columnar storage (fast aggregations)
```

---

> **DevOps focus**: S3 lifecycle policies, Velero backup/restore, storage class selection, data retention automation, cost optimization.