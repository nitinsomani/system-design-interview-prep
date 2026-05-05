# Phase 9: Monitoring & SRE — Interview Q&A

## Q1: How do you define and track SLOs?
**Answer:** 1) Identify critical user journeys (login, checkout, search). 2) Define SLIs: availability (success rate), latency (P99), throughput. 3) Set SLO targets based on user expectations and business needs: e.g., 99.9% of requests complete in <200ms. 4) Instrument with Prometheus: track `http_request_duration_seconds` histogram. 5) Calculate error budget: 1 - 0.999 = 0.1% = 43.2 min/month. 6) Dashboard showing budget consumption rate. 7) Policy: budget depleted → feature freeze, focus on reliability. Review SLOs quarterly — too many alerts = SLO too tight, never alerting = SLO too loose.

## Q2: How do you reduce alert fatigue?
**Answer:** 1) Alert on symptoms not causes (error rate >1% not CPU >80%). 2) Use SLO-based burn rate alerting: fast burn (5min window, page) + slow burn (1hr window, ticket). 3) Every alert must have a runbook and be actionable. 4) Consolidate: one alert per service per issue, not per-pod. 5) Use alert inhibition: if parent alert fires, suppress child alerts. 6) Monthly alert review: delete alerts nobody acted on. 7) Target: <2 pages per on-call shift. If more, dedicate sprint to fix root causes.

## Q3: Walk me through how you'd handle a production incident.
**Answer:** 1) **Detect**: Alert fires → acknowledge within 5 min. 2) **Triage**: Check dashboards — is it infra (USE) or app (RED)? Determine severity. 3) **Communicate**: Create incident channel, update status page within 15 min. 4) **Mitigate**: First priority is restore service (rollback, scale up, failover) — root cause analysis comes later. 5) **Resolve**: Apply fix, verify metrics return to normal. 6) **Postmortem**: Within 48 hours — blameless, timeline, root cause, what went well, 3-5 action items with owners and deadlines. Share postmortem widely. Key: mitigate first, investigate later.

## Q4: Explain the difference between monitoring and observability.
**Answer:** **Monitoring**: predefined checks — "is this metric in range?" You must know what to look for in advance. Dashboards, alerts, health checks. **Observability**: ability to understand system state from its outputs. You can investigate unknown-unknowns. Requires: high-cardinality metrics, structured logs with trace IDs, distributed traces. Example: monitoring tells you "P99 is high." Observability lets you ask "which user, from which region, hitting which endpoint, through which services, is experiencing high latency?" and drill down without deploying new code.

## Q5: How do you set up monitoring for a microservices platform?
**Answer:** Three layers: 1) **Infrastructure** (USE method): node CPU/memory/disk, pod resource usage, cluster health. Prometheus node_exporter + kube-state-metrics. 2) **Application** (RED method): per-service request rate, error rate, P99 latency. Instrument with Prometheus client libraries. 3) **Business**: order count, conversion rate, revenue. Custom metrics. Plus: distributed tracing (Tempo) with trace ID propagation for cross-service debugging. Logs (Loki) with structured JSON + trace ID correlation. Golden signals dashboard per service. Alert on SLO burn rate.

## Rapid-Fire
- **Prometheus pull vs push?** → Pull: scrapes /metrics endpoint (default). Push: via Pushgateway (short-lived jobs only)
- **Histogram vs summary?** → Histogram: server-side quantile calc, aggregatable. Summary: client-side, not aggregatable
- **Cardinality explosion?** → Too many label combinations → Prometheus OOM. Avoid high-cardinality labels (user_id)
- **Error budget burn rate?** → How fast you're consuming error budget. >1 = burning faster than allowed
- **Toil?** → Repetitive manual work that scales with service size. Automate to reduce toil
- **Runbook?** → Step-by-step guide for responding to a specific alert. Every alert needs one.