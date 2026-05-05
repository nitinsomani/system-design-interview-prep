# Phase 15: Interview Preparation — Interview Q&A

## Q1: Design a URL shortener (DevOps perspective)
**Answer:**
**Requirements**: 1000 writes/s, 100K reads/s, 99.99% availability.

**Architecture**: Client → CloudFront (cache redirects) → ALB → API (EKS) → Redis (cache) → PostgreSQL (persistence).

**Write**: POST /shorten → generate 7-char Base62 ID → store in PostgreSQL → cache in Redis → return short URL.
**Read**: GET /{id} → check Redis (99% hit) → miss: check DB → 301 redirect.

**Data**: 1000/s × 86400 × 365 = 31.5B URLs/year. 500B each = 15 TB/year → sharded PostgreSQL by hash(id).

**DevOps specifics**: Deploy on EKS with HPA (CPU + custom QPS metric). Redis cluster (6 nodes, 3 master + 3 replica). PostgreSQL RDS Multi-AZ. ArgoCD for GitOps deployment. Prometheus monitoring: redirect latency P99 < 10ms, error rate < 0.01%. Canary deployments for API changes. Velero backups daily.

## Q2: Design a notification system
**Answer:**
**Requirements**: Send push/email/SMS to 10M users, process 1000 notifications/s, at-least-once delivery.

**Architecture**: API → Kafka (order-events topic) → Notification Service → Channel Router → {Push/Email/SMS providers}.

**Components**: 1) API accepts notification request → Kafka. 2) Notification service consumes, applies user preferences (opt-out, quiet hours). 3) Channel router sends to appropriate provider (Firebase Push, SES Email, Twilio SMS). 4) DLQ for failed deliveries → retry with exponential backoff.

**DevOps**: Kafka on K8s (Strimzi, 12 partitions). KEDA auto-scale consumers on lag. Rate limiting per provider (SES: 200/s). DLQ monitoring: alert if depth > 100. Template service for email/push content. Idempotency: dedup by notification_id.

## Q3: Design a monitoring and alerting platform
**Answer:**
**Requirements**: 500 microservices, metrics + logs + traces, < 30s alert latency.

**Architecture**:
- **Metrics**: Prometheus (scrape /metrics) → Thanos (long-term, multi-cluster) → Grafana.
- **Logs**: Vector DaemonSet → Kafka → Loki → Grafana.
- **Traces**: OpenTelemetry SDK → Tempo → Grafana.
- **Alerting**: Prometheus AlertManager → PagerDuty/Slack.

**Key decisions**: Prometheus per cluster + Thanos for global view. SLO-based alerting (burn rate, not raw thresholds). Structured logs with trace ID for correlation. 15-day full-res metrics, 1-year downsampled. Alert routing: P1 → PagerDuty, P2 → Slack, P3 → ticket.

**DevOps**: kube-prometheus-stack Helm chart. Per-namespace recording rules. Dashboard-as-code (Grafana Jsonnet). Alert-as-code (PrometheusRule CRDs in Git). Cost: ~$2K/month self-hosted vs ~$15K/month Datadog.

## Q4: How would you design a disaster recovery strategy?
**Answer:**
**Classify by RPO/RTO**: Tier 1 (critical: payments) RPO=0, RTO<5min. Tier 2 (important: user service) RPO<5min, RTO<1hr. Tier 3 (internal tools) RPO<24hr, RTO<4hr.

**Tier 1**: Multi-region active-active. Aurora Global Database. DynamoDB Global Tables. Route 53 failover. Cost: 2x. **Tier 2**: Multi-AZ + cross-region read replica. Warm standby in DR region. Automated failover via Route 53. **Tier 3**: Cross-region backups. Restore from backup on demand.

**Testing**: Quarterly DR drill per tier. Chaos engineering (Litmus) for AZ failure simulation. Failover + failback procedure documented as runbook.

## Q5: Design a multi-tenant Kubernetes platform
**Answer:**
**Requirements**: 20 teams, isolation, cost attribution, self-service.

**Architecture**: Shared EKS cluster. Namespace per team. GitOps (ArgoCD AppOfApps).

**Isolation**: ResourceQuota per namespace (CPU, memory, pods). LimitRange for defaults. NetworkPolicy deny-all + allow within namespace. Pod Security Standards: restricted. RBAC: team role bound to namespace.

**Self-service**: Team creates PR to add their app → ArgoCD syncs. Helm chart library for standard patterns (web app, worker, cronjob). Internal developer portal (Backstage).

**Cost**: Kubecost per namespace → chargeback to teams. Right-sizing recommendations per team. Shared platform cost split by resource usage.

**Operations**: Centralized monitoring (Prometheus/Grafana). Per-team dashboards. Platform team manages cluster, teams manage their namespaces.

## Rapid-Fire Design Patterns
- **Rate limiter?** → Token bucket in Redis, `INCR + TTL`, return 429
- **Distributed lock?** → Redlock (Redis) or DynamoDB conditional write
- **Unique ID generator?** → Snowflake (timestamp + machine + sequence)
- **Service discovery?** → K8s DNS (service.namespace.svc.cluster.local)
- **Config management?** → ConfigMap + External Secrets + feature flags
- **API versioning?** → URL path (/v1/users) or header (Accept: v2)