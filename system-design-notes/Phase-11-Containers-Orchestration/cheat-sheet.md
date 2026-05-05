# Phase 11: Containers & Orchestration — Cheat Sheet

## Container Best Practices
```
Multi-stage builds (small images)    Non-root USER 1000
Minimal base (distroless/Alpine)     Pin versions (no :latest)
.dockerignore                        One process per container
```

## K8s Architecture
```
Control Plane: API Server → etcd, Scheduler, Controller Manager
Node: kubelet + kube-proxy + containerd
```

## Key Objects
```
Pod:        Smallest unit          Service:    Stable endpoint
Deployment: ReplicaSet manager     Ingress:    L7 HTTP routing
ConfigMap:  Config data            Secret:     Sensitive data
PVC:        Persistent storage     HPA:        Auto-scale pods
PDB:        Disruption budget      Job/CronJob: Batch/scheduled
```

## Resource Management
```
Requests: guaranteed minimum (scheduling)
Limits: hard ceiling (OOMKill if exceeded)
QoS: Guaranteed (req==limit) > Burstable > BestEffort
Always set memory limits. CPU limits optional (throttling risk).
```

## Helm Commands
```
helm install myapp ./chart -f values.yaml
helm upgrade myapp ./chart -f values.yaml
helm rollback myapp 1
helm template ./chart       # dry-run render
helm diff upgrade myapp ./chart  # preview changes
```

## Multi-Tenancy
```
Soft: Namespace + ResourceQuota + NetworkPolicy + RBAC
Hard: Separate clusters per tenant
Middle: vCluster (virtual control plane)
```