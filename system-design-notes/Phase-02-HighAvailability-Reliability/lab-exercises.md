# Phase 2: High Availability & Reliability — Lab Exercises

## Lab 1: Multi-AZ Deployment on K8s
```bash
# Deploy app with anti-affinity across zones
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ha-app
spec:
  replicas: 3
  selector:
    matchLabels: { app: ha-app }
  template:
    metadata:
      labels: { app: ha-app }
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels: { app: ha-app }
      containers:
        - name: app
          image: nginx
          ports: [{ containerPort: 80 }]
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: ha-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels: { app: ha-app }
EOF

kubectl get pods -o wide  # Verify pods are spread across zones
```

## Lab 2: Chaos — Kill Pod and Verify Recovery
```bash
# Watch pods in one terminal
kubectl get pods -w &

# Kill a pod
kubectl delete pod $(kubectl get pods -l app=ha-app -o name | head -1)

# Verify: K8s recreates the pod automatically
# Service endpoints should always show 2+ pods (PDB)
kubectl get endpoints ha-app
```

## Lab 3: DR Drill — Backup and Restore
```bash
# Simulate: backup etcd (K8s state)
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db

# Simulate: restore from backup
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db

# For databases: pg_dump / pg_restore
pg_dump -h primary -U app mydb > backup.sql
psql -h new-primary -U app mydb < backup.sql

# Measure: How long did restore take? = your actual RTO
```

## Lab 4: Availability Calculator
```
Exercise: Your system has:
  DNS (99.99%) → CDN (99.95%) → LB (99.99%) → App×3 (99.9% each) → DB Primary+Replica (99.99% each)

  App (parallel): 1-(1-0.999)^3 = 1-0.000000001 = 99.9999999%
  DB (parallel): 1-(1-0.9999)^2 = 99.999999%
  Total (series): 0.9999 × 0.9995 × 0.9999 × 1.0 × 1.0 ≈ 99.93%
  Bottleneck: CDN at 99.95% → consider multi-CDN
```