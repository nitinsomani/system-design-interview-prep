# Phase 11: Containers & Orchestration — Notes

## 1. Container Fundamentals

```
Container = process with isolated namespace + cgroup limits
  Namespace: PID, Network, Mount, UTS, IPC, User → isolation
  Cgroup: CPU, Memory, I/O limits → resource control

Docker image layers:
  FROM ubuntu:22.04          ← base layer (shared)
  RUN apt-get install nginx  ← layer 2
  COPY app /app              ← layer 3
  Each instruction = new layer, cached if unchanged

Best practices:
  - Multi-stage builds (builder → runtime, smaller images)
  - Non-root user (USER 1000)
  - Minimal base image (distroless, Alpine)
  - Pin versions (FROM nginx:1.25.3, not :latest)
  - .dockerignore to exclude unnecessary files
  - One process per container
```

## 2. Kubernetes Architecture

```
Control Plane:
  API Server:        Frontend, all operations go through here
  etcd:              Distributed KV store (cluster state)
  Scheduler:         Assigns pods to nodes (resource-aware)
  Controller Manager: Reconciliation loops (desired → actual)

Node:
  kubelet:           Manages pods on the node
  kube-proxy:        Network rules (iptables/IPVS)
  Container Runtime: containerd (runs containers)

Key objects:
  Pod:         Smallest deployable unit (1+ containers)
  Deployment:  Manages ReplicaSets, rolling updates
  Service:     Stable network endpoint (ClusterIP, NodePort, LB)
  Ingress:     L7 HTTP routing
  ConfigMap:   Non-sensitive configuration
  Secret:      Sensitive data (base64, not encrypted by default!)
  PVC:         Persistent storage claim
  HPA:         Horizontal pod autoscaler
  PDB:         Pod disruption budget
```

## 3. Resource Management

```
Requests vs Limits:
  Requests: guaranteed minimum (scheduler uses for placement)
  Limits: maximum allowed (OOMKill if exceeded for memory)

  resources:
    requests: { cpu: 100m, memory: 128Mi }   # Scheduling guarantee
    limits: { cpu: 500m, memory: 512Mi }      # Hard ceiling

Best practices:
  - Always set requests (otherwise: BestEffort QoS, first to be evicted)
  - CPU limit debate: may cause throttling → some teams don't set CPU limits
  - Memory limit: always set (OOM is better than node pressure)
  - Right-size: use VPA recommendations or metrics analysis

QoS Classes:
  Guaranteed:  requests == limits for all containers
  Burstable:   requests < limits (or only requests set)
  BestEffort:  no requests or limits → first evicted
```

## 4. Helm Charts

```
Package manager for K8s:
  Chart: bundle of templates + values
  Release: installed instance of a chart
  Repository: collection of charts

Structure:
  mychart/
    Chart.yaml          # metadata
    values.yaml         # default config
    templates/
      deployment.yaml   # {{ .Values.image.tag }}
      service.yaml
      ingress.yaml
      _helpers.tpl      # template helpers

Key commands:
  helm install myapp ./mychart -f prod-values.yaml
  helm upgrade myapp ./mychart -f prod-values.yaml
  helm rollback myapp 1
  helm list
  helm template ./mychart  # render without installing (dry-run)

Best practices:
  - Use values.yaml for environment differences
  - Pin chart versions in requirements
  - Use helm diff plugin before upgrade
  - Store charts in OCI registry (ECR, Harbor)
```

## 5. K8s Operators

```
Operator = custom controller + CRD
  Extends K8s API for complex applications

Example: PostgreSQL Operator (CloudNativePG)
  Create PostgreSQL cluster with:
    apiVersion: postgresql.cnpg.io/v1
    kind: Cluster
    spec:
      instances: 3
      storage: { size: 100Gi }
  
  Operator handles: replication, failover, backup, monitoring

Common operators:
  Databases:    CloudNativePG, Percona, Strimzi (Kafka)
  Monitoring:   Prometheus Operator (kube-prometheus-stack)
  Certificates: cert-manager
  Secrets:      External Secrets Operator
```

## 6. Multi-Tenancy

```
Soft multi-tenancy (namespaces):
  - Namespace per team/environment
  - ResourceQuota: limit total resources per namespace
  - LimitRange: default resource limits per pod
  - NetworkPolicy: restrict inter-namespace traffic
  - RBAC: role bindings per namespace

Hard multi-tenancy (cluster per tenant):
  - Complete isolation
  - Higher cost, more operational overhead
  - Required for strict compliance (some financial/healthcare)

Virtual clusters (vCluster):
  - Virtual control plane per tenant in shared cluster
  - Middle ground: strong isolation, lower cost
```

---

> **DevOps focus**: Image optimization, resource management, Helm lifecycle, operator patterns, namespace isolation, cluster operations.