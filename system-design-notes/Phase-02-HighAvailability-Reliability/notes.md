# Phase 2: High Availability & Reliability — Notes

## 1. Redundancy Patterns

```
Active-Active:
  Both instances handle traffic simultaneously
  LB distributes across both
  If one fails → other handles all traffic (capacity planning needed)
  Example: 2 API server pods behind K8s Service

Active-Passive (Hot Standby):
  Primary handles all traffic
  Standby is running but idle (receives replicated data)
  Failover: Standby promoted on primary failure
  Example: RDS Multi-AZ (synchronous replication, auto-failover)

Active-Passive (Warm Standby):
  Standby is running at reduced capacity
  Takes minutes to scale up during failover
  Example: DR region with minimal infrastructure

Active-Passive (Cold Standby):
  Standby is OFF, data backed up periodically
  Takes hours to bring up
  Example: Restore from S3 backup to new region
```

## 2. Replication

```
Synchronous Replication:
  Write → Primary → Replica (wait for ACK) → confirm to client
  Pros: No data loss (RPO = 0)
  Cons: Higher latency (wait for replica), lower throughput
  Use: Financial data, critical transactions

Asynchronous Replication:
  Write → Primary (confirm immediately) → Replica (later)
  Pros: Low latency, high throughput
  Cons: Data loss possible if primary fails before replication (RPO > 0)
  Use: Read replicas, cross-region replication

Semi-synchronous:
  Write → Primary → at least 1 replica ACK → confirm
  Balance between consistency and performance
  Use: MySQL semi-sync, PostgreSQL synchronous_standby_names
```

## 3. Failover Strategies

```
DNS Failover:
  Route 53 health checks → remove unhealthy endpoint
  TTL-dependent: 60-300s propagation delay
  Use: Region-level failover

Database Failover:
  RDS Multi-AZ: automatic, 1-2 min, DNS endpoint unchanged
  PostgreSQL Patroni: automatic leader election via etcd
  Redis Sentinel: monitors primary, promotes replica

Application Failover:
  Circuit breaker: stops calling failed dependency
  Retry with fallback: try primary → backup → cached response
  Load balancer health checks: remove unhealthy backends

Stateful Failover Challenges:
  In-flight transactions → may be lost
  Connection state → clients must reconnect
  Replication lag → may lose recent writes
  Split-brain → two nodes think they're primary
    Prevention: Fencing (STONITH), quorum-based decisions
```

## 4. Disaster Recovery

```
RPO (Recovery Point Objective):
  Maximum acceptable data loss
  RPO = 0: No data loss (synchronous replication)
  RPO = 1 hour: Up to 1 hour of data loss (hourly backups)
  RPO = 24 hours: Daily backups acceptable

RTO (Recovery Time Objective):
  Maximum acceptable downtime
  RTO = 0: No downtime (active-active multi-region)
  RTO = 15 min: Quick failover (hot standby)
  RTO = 4 hours: Restore from backup

DR Strategies (cost: low → high):
  Backup & Restore:    RPO: hours, RTO: hours, Cost: $
  Pilot Light:         RPO: minutes, RTO: minutes, Cost: $$
  Warm Standby:        RPO: seconds, RTO: seconds, Cost: $$$
  Multi-Region Active: RPO: ~0, RTO: ~0, Cost: $$$$
```

## 5. Chaos Engineering

```
Principle: Proactively inject failures to find weaknesses

Tools:
  Chaos Monkey (Netflix): Kill random instances
  Litmus (K8s): Pod kill, network chaos, disk fill
  Gremlin: Commercial, comprehensive failure injection
  AWS FIS: Managed fault injection service
  Toxiproxy: Network condition simulation

Experiments:
  1. Kill a pod → does traffic failover?
  2. Increase latency → does circuit breaker trigger?
  3. Fill disk → does alert fire? Does app handle gracefully?
  4. Kill a node → do pods reschedule?
  5. Block network to DB → does app show degraded mode?

Process:
  1. Define steady state (normal metrics)
  2. Hypothesize: "If X fails, system should..."
  3. Inject failure in staging first
  4. Monitor: did system behave as expected?
  5. Fix gaps → repeat in production (with blast radius limits)
```

---

> **Key DevOps takeaway**: HA is about eliminating SPOFs through redundancy, automated failover, and regular DR testing.