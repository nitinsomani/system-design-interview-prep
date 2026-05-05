# Phase 9: Monitoring & SRE Practices — Notes

## 1. Observability Pillars

```
Metrics:     Numeric values over time (CPU, latency, error rate)
             Tools: Prometheus, CloudWatch, Datadog
             Best for: Alerting, dashboards, trends

Logs:        Discrete events with context
             Tools: Loki, Elasticsearch, CloudWatch Logs
             Best for: Debugging, audit trail, search

Traces:      Request flow across services
             Tools: Tempo, Jaeger, X-Ray
             Best for: Latency analysis, dependency mapping

Profiling:   CPU/memory usage at code level (newest pillar)
             Tools: Pyroscope, Parca
             Best for: Performance optimization
```

## 2. SLI / SLO / SLA

```
SLI (Service Level Indicator):
  Measurable metric: latency, error rate, throughput
  Example: P99 latency of /api/orders endpoint

SLO (Service Level Objective):
  Target for SLI: P99 latency < 200ms for 99.9% of requests
  Internal goal, set by engineering team

SLA (Service Level Agreement):
  Contract with customers: 99.9% uptime or credit issued
  SLA should be less strict than SLO (buffer)

  SLO: 99.95% → SLA: 99.9% → buffer for safe operation

Error Budget:
  Budget = 1 - SLO = allowed downtime/errors
  99.9% SLO → 0.1% error budget → 43.2 min/month
  
  When budget is consumed:
    - Freeze feature releases
    - Focus on reliability work
    - Tighten change management

Error budget policy:
  > 50% remaining:  Normal development
  25-50% remaining: Increase review rigor
  < 25% remaining:  Feature freeze, reliability focus
  0% remaining:     Incident mode, only reliability fixes
```

## 3. Prometheus + Grafana Stack

```
Architecture:
  App → /metrics endpoint (instrumented)
  Prometheus → scrapes /metrics every 15-30s
  PromQL → query language for metrics
  Alertmanager → routes alerts (PagerDuty, Slack)
  Grafana → visualization dashboards

Key metric types:
  Counter:    Monotonically increasing (requests_total)
  Gauge:      Goes up/down (memory_usage_bytes)
  Histogram:  Distribution (request_duration_seconds)
  Summary:    Pre-calculated quantiles

Essential PromQL:
  Rate:        rate(http_requests_total[5m])
  Error rate:  rate(http_errors_total[5m]) / rate(http_requests_total[5m])
  P99 latency: histogram_quantile(0.99, rate(http_duration_bucket[5m]))
  Saturation:  container_memory_usage_bytes / container_spec_memory_limit_bytes

RED method (request-driven services):
  Rate:    Requests per second
  Errors:  Error rate
  Duration: Latency distribution

USE method (infrastructure):
  Utilization: % resource used
  Saturation:  Queue depth / waiting
  Errors:      Error count
```

## 4. Alerting Best Practices

```
Alert on symptoms, not causes:
  Bad:   CPU > 80%  (cause — may not affect users)
  Good:  P99 latency > 500ms  (symptom — users affected)
  Good:  Error rate > 1%  (symptom — users affected)

Severity levels:
  P1 (Critical):  Revenue impact, page on-call immediately
  P2 (High):      Degraded service, page within 15 min
  P3 (Medium):    Non-critical, Slack notification, fix next business day
  P4 (Low):       Cosmetic, ticket, fix in sprint

Reduce alert fatigue:
  - Alert on SLO burn rate, not raw metrics
  - Use multi-window burn rate (fast 5min + slow 1hr)
  - Require sustained condition (for: 5m in Prometheus)
  - Every alert must have a runbook
  - Review alerts monthly: delete noisy ones
```

## 5. Incident Management

```
Lifecycle:
  Detect → Triage → Mitigate → Resolve → Postmortem

Roles:
  Incident Commander: coordinates response
  Communication Lead: updates stakeholders
  Technical Lead: drives diagnosis and fix

Severity classification:
  SEV1: Service down, all users affected
  SEV2: Major feature broken, many users affected
  SEV3: Minor feature broken, some users affected

Communication:
  Internal: Slack incident channel (#inc-YYMMDD-title)
  External: Status page update within 15 min (Statuspage.io)
  Cadence: Update every 30 min until resolved

Postmortem (blameless):
  - Timeline of events
  - What went wrong (root cause)
  - What went right (what helped)
  - Action items with owners and deadlines
  - Share widely (learn from failures)
```

## 6. On-Call Best Practices

```
Rotation:     Weekly, 2-person (primary + secondary)
Handoff:      Document ongoing issues, share context
Escalation:   Primary (5 min) → Secondary (15 min) → Manager
Compensation: Extra pay or time off for on-call duty

On-call hygiene:
  - < 2 pages per shift (if more, fix root causes)
  - Every page must be actionable
  - Runbooks for every alert
  - Follow-the-sun for global teams
```

---

> **DevOps focus**: Prometheus/Grafana setup, SLO definition, error budget tracking, alert tuning, incident response, blameless postmortems.