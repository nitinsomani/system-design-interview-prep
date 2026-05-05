# Phase 11: Containers & Orchestration — Lab Exercises

## Lab 1: Multi-Stage Docker Build
```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o server .

# Stage 2: Runtime
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app/server /server
USER 65534:65534
EXPOSE 8080
ENTRYPOINT ["/server"]
```
```bash
# Build and compare sizes
docker build -t myapp:multi .
docker images myapp
# Single stage: ~800MB  vs  Multi-stage: ~15MB
```

## Lab 2: Namespace Isolation (Multi-Tenancy)
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata: { name: team-alpha }
---
apiVersion: v1
kind: ResourceQuota
metadata: { name: quota, namespace: team-alpha }
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "20"
---
apiVersion: v1
kind: LimitRange
metadata: { name: defaults, namespace: team-alpha }
spec:
  limits:
    - default: { cpu: 500m, memory: 256Mi }
      defaultRequest: { cpu: 100m, memory: 128Mi }
      type: Container
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: deny-all, namespace: team-alpha }
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
  ingress:
    - from: [{ podSelector: {} }]
  egress:
    - to: [{ podSelector: {} }]
    - to: [{ namespaceSelector: { matchLabels: { name: kube-system } } }]
      ports: [{ port: 53, protocol: UDP }]
EOF
```

## Lab 3: Pod Topology Spread
```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata: { name: web }
spec:
  replicas: 6
  selector: { matchLabels: { app: web } }
  template:
    metadata: { labels: { app: web } }
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector: { matchLabels: { app: web } }
      containers:
        - name: web
          image: nginx
          resources:
            requests: { cpu: 100m, memory: 64Mi }
EOF
# Pods spread evenly across AZs: 2 per zone (3 zones)
kubectl get pods -l app=web -o wide
```

## Lab 4: Resource Right-Sizing Exercise
```
Given Prometheus metrics for "order-service" (7-day data):

  CPU usage:
    P50: 50m    P95: 150m    P99: 300m    Peak: 500m
  Memory usage:
    P50: 200Mi  P95: 350Mi   P99: 400Mi   Peak: 450Mi
  Current settings:
    requests: { cpu: 1000m, memory: 1Gi }
    limits: { cpu: 2000m, memory: 2Gi }

  Analysis:
    CPU request 1000m but P95 is 150m → 6.6x over-provisioned
    Memory request 1Gi but P95 is 350Mi → 2.8x over-provisioned

  Recommended:
    requests: { cpu: 200m, memory: 400Mi }
    limits: { cpu: 500m, memory: 600Mi }

  Savings per pod: 800m CPU + 600Mi memory
  With 10 replicas: 8 CPU cores + 6Gi memory freed
```