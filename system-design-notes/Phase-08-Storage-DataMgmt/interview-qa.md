# Phase 8: Storage & Data Management — Interview Q&A

## Q1: How do you choose between S3, EBS, and EFS?
**Answer:** **EBS**: database storage (PostgreSQL data dir), low latency (<1ms), single instance attachment. Use gp3 for general, io2 for high-IOPS DB. **EFS**: shared filesystem across pods/instances (CMS uploads, shared config). Higher latency but concurrent access. **S3**: everything else — backups, logs, static assets, data lake. Unlimited scale, cheapest, 11 nines durability. Rule: if it's a database → EBS. If shared file access → EFS. Everything else → S3.

## Q2: How would you design a cost-effective storage strategy for a growing application?
**Answer:** Data tiering with lifecycle policies: 1) Active data in S3 Standard (first 30 days). 2) Infrequent access → S3-IA (30-90 days). 3) Archive → Glacier (90+ days). 4) Delete after retention period. For logs: ship to S3 via Fluentd/Vector, query with Athena (pay per query, no infra). For databases: RDS automated snapshots (35 days) + manual snapshots to S3. For K8s PVs: Velero daily backups to S3. Monitor with Cost Explorer, set budget alerts. This typically reduces storage costs by 60-80%.

## Q3: Explain your backup and disaster recovery strategy.
**Answer:** 3-2-1 rule: 3 copies, 2 media, 1 offsite. Implementation: 1) RDS: automated daily snapshots + point-in-time recovery (5-min granularity) + cross-region snapshot copy. 2) K8s: Velero scheduled backups (resources + PV snapshots) to S3 in different region. 3) Application data (S3): cross-region replication + versioning. Recovery testing: quarterly DR drills — restore to separate environment, verify data integrity. RPO: 5 min (DB), 24 hours (K8s resources). RTO: 1 hour (DB failover), 4 hours (full cluster restore).

## Q4: How do you handle data migration with zero downtime?
**Answer:** Dual-write pattern: 1) Deploy new storage alongside old. 2) Application writes to both old and new. 3) Backfill historical data (batch job). 4) Verify consistency (compare checksums). 5) Switch reads to new storage. 6) Stop writes to old, decommission. For databases: AWS DMS for continuous replication. For S3: S3 Batch Operations or custom Lambda pipeline. Key: always have rollback plan — keep old storage running until new is proven.

## Q5: How would you design a log aggregation pipeline?
**Answer:** Sources → Collection → Transport → Storage → Query. 1) **Sources**: app logs (stdout), system logs, audit logs. 2) **Collection**: Fluentd/Vector DaemonSet on each node. 3) **Transport**: Kafka (buffer, fan-out to multiple consumers). 4) **Storage**: S3 in Parquet format (compressed, columnar). Partitioned by date: s3://logs/year=2024/month=01/day=15/. 5) **Query**: Athena for ad-hoc, Loki/Elasticsearch for real-time search. Retention: 30 days in Elasticsearch (hot), 1 year in S3 (cold). Cost: ~$0.005/GB/month in S3 vs $30/GB/month in Elasticsearch.

## Rapid-Fire
- **S3 consistency model?** → Strong read-after-write consistency (since Dec 2020)
- **S3 Transfer Acceleration?** → Uses CloudFront edge for faster cross-region uploads
- **EBS vs instance store?** → EBS persists, instance store is ephemeral (lost on stop)
- **Velero?** → K8s backup tool: resources + PV snapshots → S3, scheduled/on-demand
- **Parquet vs JSON?** → Parquet: columnar, compressed, 10x smaller, 100x faster queries
- **S3 versioning?** → Keep all versions of objects, protect against accidental deletes