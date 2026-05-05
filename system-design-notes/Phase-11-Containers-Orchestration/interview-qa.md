# Phase 11: Containers & Orchestration — Interview Q&A

## Q1: How does Kubernetes scheduling work?
**Answer:** Scheduler watches for unscheduled pods. Process: 1) **Filtering**: eliminate nodes that don't meet requirements (resource requests, node selectors, taints/tolerations, affinity rules, PV availability). 2) **Scoring**: rank remaining nodes by criteria (least requested resources, pod spreading, node affinity weight). 3) **Binding**: assign pod to highest-scoring node. Key controls: nodeSelector (simple), nodeAffinity (flexible), podAntiAffinity (spread across zones), taints/tolerations (dedicated nodes), topologySpreadConstraints (even distribution). Resource requests are critical — without them, scheduler can't make informed decisions.

## Q2: How do you right-size container resources?
**Answer:** 1) Start with reasonable defaults (100m CPU, 128Mi memory). 2) Deploy with Prometheus metrics collection. 3) After 1-2 weeks, analyze actual usage: `container_cpu_usage_seconds_total`, `container_memory_working_set_bytes`. 4) Set requests to P95 usage, limits to P99 peak. 5) Use VPA (Vertical Pod Autoscaler) in recommendation mode — it analyzes usage and suggests values. 6) Iterate quarterly. Watch for: CPU throttling (limits too low), OOMKills (memory limit too low), wasted resources (requests too high). Rule of thumb: requests:limits ratio of 1:2 for CPU, 1:1.5 for memory.

## Q3: How would you design a multi-tenant Kubernetes platform?
**Answer:** Isolation layers: 1) **Namespace per tenant** with ResourceQuota (max CPU, memory, PVCs, pods). 2) **LimitRange** for default requests/limits per pod. 3) **NetworkPolicy**: deny all ingress by default, allow only within namespace. 4) **RBAC**: tenant-scoped roles (edit within namespace, no cluster-admin). 5) **Pod Security Standards**: restricted (no root, no host network). 6) **Separate node pools** for noisy-neighbor isolation (optional). 7) **Monitoring**: per-namespace cost allocation (Kubecost). For stronger isolation: vCluster gives each tenant a virtual control plane while sharing worker nodes.

## Q4: Explain the difference between StatefulSet and Deployment.
**Answer:** **Deployment**: stateless workloads, pods are interchangeable, random names (app-xyz123), parallel scaling, any pod can be killed/replaced. **StatefulSet**: stateful workloads, ordered creation/deletion (app-0, app-1, app-2), stable network identity (headless service), persistent volume per pod (not shared), ordered rolling updates. Use StatefulSet for: databases (PostgreSQL, MySQL), Kafka brokers, Elasticsearch nodes, ZooKeeper — anything needing stable identity or dedicated storage. Prefer operators over raw StatefulSets for databases (they handle failover, backup, etc.).

## Q5: How do you handle secrets in Kubernetes?
**Answer:** Default K8s secrets are base64-encoded (NOT encrypted) — insufficient for production. Solutions: 1) **External Secrets Operator**: syncs secrets from AWS Secrets Manager/Vault → K8s Secret automatically. 2) **Sealed Secrets**: encrypt secrets client-side, store encrypted in Git, controller decrypts in-cluster. 3) **HashiCorp Vault**: CSI driver mounts secrets as files, or sidecar injector. 4) **SOPS**: encrypt secret YAML files with KMS key, decrypt in CI/CD. Best practice: never store plain secrets in Git, rotate regularly, audit access, use short-lived credentials where possible (IRSA for AWS).

## Rapid-Fire
- **Init container?** → Runs before app containers, for setup tasks (DB migration, config)
- **Sidecar?** → Helper container in same pod (log shipper, proxy, secret injector)
- **PDB?** → Pod Disruption Budget: min available pods during voluntary disruption
- **Taint vs toleration?** → Taint: node repels pods. Toleration: pod accepts taint.
- **Headless service?** → ClusterIP: None. Returns pod IPs directly. For StatefulSets.
- **Ephemeral containers?** → Debug running pods: kubectl debug pod/app -it --image=busybox