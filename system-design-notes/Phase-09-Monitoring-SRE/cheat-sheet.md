# Phase 9: Monitoring & SRE — Cheat Sheet

## Observability Stack
```
Metrics:  Prometheus → Grafana        (alerting, trends)
Logs:     Loki / Elasticsearch        (debugging, search)
Traces:   Tempo / Jaeger              (latency, dependencies)
```

## SLI / SLO / SLA
```
SLI: What you measure (P99 latency, error rate)
SLO: Internal target (99.9% requests < 200ms)
SLA: Customer contract (99.9% uptime or credit)
Error budget: 1 - SLO (99.9% → 43.2 min/month downtime allowed)
```

## PromQL Essentials
```
Rate:        rate(requests_total[5m])
Error %:     rate(errors[5m]) / rate(requests[5m]) * 100
P99:         histogram_quantile(0.99, rate(duration_bucket[5m]))
Saturation:  memory_usage / memory_limit
```

## RED Method (Services)
```
Rate:      Requests/sec
Errors:    Error rate %
Duration:  P50, P95, P99 latency
```

## USE Method (Infra)
```
Utilization: % used (CPU, memory, disk)
Saturation:  Queue depth, throttling
Errors:      Hardware/software errors
```

## Alerting Rules
```
Alert on symptoms (P99 > 500ms), NOT causes (CPU > 80%)
Multi-window burn rate for SLO alerts
Every alert needs: runbook + severity + owner
Target: < 2 pages per on-call shift
```

## Incident Response
```
Detect → Triage → Mitigate → Resolve → Postmortem
SEV1: all users affected → page immediately
Status page update: within 15 min
Postmortem: blameless, timeline, action items with owners
```