# Phase 2: High Availability & Reliability — Cheat Sheet

## Redundancy Patterns
```
Active-Active:   Both serve traffic (K8s pods, multi-region)
Active-Passive:  Standby promoted on failure (RDS Multi-AZ)
Hot Standby:     Running, ready instantly
Warm Standby:    Running at reduced capacity, minutes to scale
Cold Standby:    Off, hours to restore from backup
```

## Replication
```
Synchronous:  Write confirmed after replica ACK (RPO=0, slower)
Asynchronous: Write confirmed immediately (RPO>0, faster)
Semi-sync:    At least 1 replica ACK (balanced)
```

## DR Strategy
```
Strategy          RPO         RTO          Cost
Backup/Restore    hours       hours        $
Pilot Light       minutes     minutes      $$
Warm Standby      seconds     seconds      $$$
Active-Active     ~0          ~0           $$$$
```

## Failover Checklist
```
✓ Health checks on every component
✓ Auto-failover for databases (RDS Multi-AZ, Patroni)
✓ Circuit breakers for inter-service calls
✓ DNS failover (Route 53 health checks)
✓ PDB (Pod Disruption Budget) in K8s
✓ Anti-affinity: spread pods across nodes/AZs
✓ Regular DR drills (quarterly minimum)
```

## Chaos Engineering
```
Kill pod → Litmus      Kill node → AWS FIS
Add latency → Toxiproxy    Fill disk → stress-ng
Block network → iptables   Corrupt DNS → CoreDNS override
```