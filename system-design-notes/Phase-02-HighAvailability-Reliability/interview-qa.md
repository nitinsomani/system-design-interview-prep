# Phase 2: High Availability & Reliability — Interview Q&A

## Q1: How do you design a system for 99.99% availability?
**Answer:** Multi-AZ deployment with no SPOFs. Database: RDS Multi-AZ (auto-failover). App: 3+ pods across AZs with HPA. LB: cloud-managed (ALB). DNS: Route 53 with health checks. Circuit breakers prevent cascade failures. Pod anti-affinity spreads replicas. PDB ensures minimum pods during maintenance. Monitoring: alerting on error rate, latency — MTTR < 5 min via runbooks.

## Q2: Explain RPO and RTO with real examples.
**Answer:** RPO = max data loss tolerable. RTO = max downtime tolerable. Banking app: RPO=0 (synchronous replication), RTO=1 min (hot standby). Blog platform: RPO=1 hour (hourly snapshots), RTO=30 min (restore from backup). I match DR strategy to business need: active-active for RPO/RTO~0, pilot light for minutes, backup/restore for hours.

## Q3: What is split-brain and how do you prevent it?
**Answer:** Split-brain: two nodes both think they're primary after network partition — both accept writes → data divergence. Prevention: quorum-based consensus (need majority to be primary — 3 nodes need 2 votes), fencing/STONITH (kill the other node), leader lease with timeout. In K8s: etcd uses Raft consensus (requires quorum). PostgreSQL Patroni uses etcd for leader election.

## Q4: How does chaos engineering improve reliability?
**Answer:** Proactively inject failures to find weaknesses BEFORE they hit production. Process: define steady state metrics → hypothesize behavior under failure → inject (pod kill, network latency, disk fill) → verify system self-heals → fix gaps. Start in staging, then production with blast radius limits. Tools: Litmus (K8s), Gremlin, AWS FIS. Example: killed a DB primary in staging, discovered failover took 5 min instead of expected 30s — fixed Patroni config.

## Q5: Active-active vs active-passive — when to use each?
**Answer:** Active-active: both regions serve traffic, lower latency for global users, higher cost, complex data sync (conflict resolution needed). Active-passive: one region primary, DR region idle — simpler, cheaper, but RTO > 0. I use active-active for global latency-sensitive apps (e-commerce) and active-passive for cost-sensitive internal tools. Key challenge in active-active: data consistency across regions.

## Rapid-Fire
- **MTBF?** → Mean Time Between Failures
- **MTTR?** → Mean Time To Recovery
- **STONITH?** → Shoot The Other Node In The Head (fencing)
- **Quorum?** → Majority of nodes must agree (N/2 + 1)
- **Health check types?** → TCP, HTTP, gRPC, command exec
- **PDB?** → Pod Disruption Budget — min pods during voluntary disruption
- **Failover time for RDS Multi-AZ?** → 1-2 minutes
- **What causes split-brain?** → Network partition + no quorum/fencing