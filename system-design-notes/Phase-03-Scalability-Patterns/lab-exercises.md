# Phase 3: Scalability Patterns — Lab Exercises

## Lab 1: HPA Auto-Scaling
```bash
# Deploy app with resource requests
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata: { name: scale-app }
spec:
  replicas: 2
  selector: { matchLabels: { app: scale-app } }
  template:
    metadata: { labels: { app: scale-app } }
    spec:
      containers:
        - name: app
          image: nginx
          resources:
            requests: { cpu: 100m, memory: 64Mi }
            limits: { cpu: 200m, memory: 128Mi }
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: scale-app }
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: scale-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target: { type: Utilization, averageUtilization: 50 }
EOF

# Generate load → watch HPA scale up
kubectl run load-gen --image=busybox --rm -it -- sh -c \
  "while true; do wget -qO- http://scale-app; done"

# Watch scaling
kubectl get hpa -w
```

## Lab 2: Stateless Design Verification
```bash
# Test: kill a pod and verify service continues
kubectl delete pod $(kubectl get pod -l app=scale-app -o name | head -1)
kubectl exec deploy/load-gen -- wget -qO- http://scale-app/
# Service continues without interruption = stateless ✓
```

## Lab 3: Scaling Estimation Exercise
```
Scenario: Design a URL shortener
  Write: 1000/sec   Read: 100,000/sec   URL size: 500 bytes

  API servers: 100K/50K per server = 2 servers (+ redundancy = 4)
  Cache: 20% of 100K × 500B × 86400 = 864 GB/day → Redis cluster 200GB
  DB writes: 1000/sec × 500B = 500 KB/s → single PostgreSQL handles this
  DB reads: offloaded to cache → DB only sees cache misses
  Storage: 1000/sec × 500B × 86400 × 365 × 5yr = 27 TB → S3 or sharded DB
```

## Lab 4: Connection Pooling Test
```bash
# Deploy PgBouncer as sidecar (concept)
# pgbouncer.ini:
#   [databases]
#   mydb = host=postgres port=5432 dbname=mydb
#   [pgbouncer]
#   pool_mode = transaction
#   max_client_conn = 1000
#   default_pool_size = 20

# Monitor connections:
# SELECT count(*) FROM pg_stat_activity;
# Without PgBouncer: 500 pods × 20 conn = 10,000
# With PgBouncer: 500 pods → PgBouncer → 20 DB connections
```