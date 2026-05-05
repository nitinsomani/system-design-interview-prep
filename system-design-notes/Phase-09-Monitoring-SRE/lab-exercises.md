# Phase 9: Monitoring & SRE — Lab Exercises

## Lab 1: Prometheus + Grafana on K8s
```bash
# Install kube-prometheus-stack (Helm)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace \
  --set grafana.adminPassword=admin123

# Access Grafana
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
# Open http://localhost:3000 → admin/admin123

# Key dashboards to import:
# - K8s Cluster Overview (ID: 6417)
# - Node Exporter Full (ID: 1860)
# - K8s Pod Resources (ID: 6879)

# PromQL practice:
# CPU usage: rate(container_cpu_usage_seconds_total{namespace="production"}[5m])
# Memory: container_memory_usage_bytes{namespace="production"}
# Pod restarts: kube_pod_container_status_restarts_total
```

## Lab 2: SLO-Based Alerting
```yaml
# PrometheusRule for SLO burn rate alerting
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: api-slo-alerts
  namespace: monitoring
spec:
  groups:
    - name: slo.rules
      rules:
        # Error rate SLI
        - record: sli:http_error_rate:5m
          expr: |
            rate(http_requests_total{status=~"5.."}[5m])
            / rate(http_requests_total[5m])
        # Fast burn (page): 14.4x budget in 1 hour
        - alert: HighErrorBurnRate
          expr: sli:http_error_rate:5m > (14.4 * 0.001)
          for: 2m
          labels: { severity: critical }
          annotations:
            summary: "Error budget burning 14.4x faster than allowed"
            runbook: "https://wiki/runbooks/high-error-rate"
        # Slow burn (ticket): 3x budget in 6 hours
        - alert: ElevatedErrorRate
          expr: sli:http_error_rate:5m > (3 * 0.001)
          for: 30m
          labels: { severity: warning }
```

## Lab 3: Incident Response Drill
```
Scenario: API latency spike at 2 AM

  1. Alert fires: "P99 latency > 500ms for api-service"
     → Acknowledge in PagerDuty

  2. Triage (5 min):
     - Check Grafana dashboard: P99 jumped from 100ms to 2s
     - Check error rate: normal (not errors, just slow)
     - Check infrastructure: CPU normal, memory normal
     - Check recent deployments: deploy at 1:55 AM ← suspicious

  3. Mitigate (15 min):
     - Rollback deployment: kubectl rollout undo deploy/api-service
     - Verify: P99 drops back to 100ms ← confirmed

  4. Communicate:
     - Status page: "API latency resolved, investigating root cause"
     - Slack #incidents: timeline + rollback confirmed

  5. Postmortem (next day):
     Root cause: New DB query without index (table scan)
     Action items:
     - Add EXPLAIN ANALYZE to PR review checklist
     - Add slow query alerting (queries > 100ms)
     - Add load testing to CI pipeline
```

## Lab 4: Error Budget Calculator
```
Exercise: Calculate error budget consumption

  SLO: 99.9% availability
  Month: 30 days = 43,200 minutes
  Budget: 43,200 × 0.001 = 43.2 minutes of downtime

  Incidents this month:
    Week 1: 5 min outage (deploy rollback)
    Week 2: 0 min
    Week 3: 15 min partial outage (DB failover)
    Week 4: 2 min (DNS propagation)

  Total consumed: 22 min / 43.2 min = 50.9%
  Remaining: 49.1%

  Decision: 25-50% range → increase review rigor
    - Require extra approval for production deploys
    - Add integration tests before deploy
    - Continue feature work with caution
```