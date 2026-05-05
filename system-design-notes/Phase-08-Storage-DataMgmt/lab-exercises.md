# Phase 8: Storage & Data Management — Lab Exercises

## Lab 1: S3 Lifecycle Policy
```bash
# Create lifecycle rule (AWS CLI)
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-app-data \
  --lifecycle-configuration '{
    "Rules": [
      {
        "ID": "TierAndExpire",
        "Status": "Enabled",
        "Filter": { "Prefix": "logs/" },
        "Transitions": [
          { "Days": 30, "StorageClass": "STANDARD_IA" },
          { "Days": 90, "StorageClass": "GLACIER" }
        ],
        "Expiration": { "Days": 365 }
      }
    ]
  }'

# Verify
aws s3api get-bucket-lifecycle-configuration --bucket my-app-data

# Cost calculation:
# 1 TB logs/month, stored 1 year:
#   Without lifecycle: 12 TB × $0.023 = $276/month
#   With lifecycle: 1TB×$0.023 + 2TB×$0.0125 + 9TB×$0.004 = $84/month
#   Savings: 70%
```

## Lab 2: Velero Backup & Restore
```bash
# Install Velero
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.8.0 \
  --bucket velero-backups \
  --backup-location-config region=us-east-1 \
  --snapshot-location-config region=us-east-1

# Create scheduled backup
velero schedule create daily-backup \
  --schedule="0 2 * * *" \
  --include-namespaces production \
  --ttl 720h

# Manual backup
velero backup create pre-migration --include-namespaces production

# Check backup status
velero backup describe pre-migration

# Restore to different namespace (DR drill)
velero restore create --from-backup pre-migration \
  --namespace-mappings production:dr-test

# Verify restoration
kubectl -n dr-test get all
```

## Lab 3: K8s Storage Classes
```bash
# Create storage classes for different workloads
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: { name: fast-ssd }
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: { name: high-iops }
provisioner: ebs.csi.aws.com
parameters:
  type: io2
  iops: "10000"
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
EOF

# Use in PVC:
# storageClassName: fast-ssd   → general DB
# storageClassName: high-iops  → heavy transactional DB
```

## Lab 4: Storage Design Exercise
```
Scenario: Video streaming platform
  Videos: 10K new/day, average 500MB each
  Thumbnails: 10K new/day, 100KB each
  Metadata: 10K records/day, 5KB each
  User activity: 10M events/day, 1KB each

  Storage design:
  1. Videos: S3 Standard → S3-IA (30d) → Glacier (1yr)
     Daily: 10K × 500MB = 5 TB/day
     Monthly: 150 TB → lifecycle to reduce cost

  2. Thumbnails: S3 + CloudFront CDN
     Cache-Control: public, max-age=86400
     Small, frequently accessed → keep in Standard

  3. Metadata: PostgreSQL (RDS)
     10K/day = small, needs queries + joins
     Daily backup + 7-day PITR

  4. User activity: Kafka → S3 (Parquet, partitioned by date)
     Query via Athena for analytics
     10M × 1KB = 10 GB/day → cheap in S3

  Total monthly cost estimate:
    S3: 150TB × $0.023 = $3,450 (before lifecycle)
    RDS: db.r6g.large = $300
    CloudFront: 1 PB transfer = $40,000
    Total: ~$44,000/month
```