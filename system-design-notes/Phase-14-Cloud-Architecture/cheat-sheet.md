# Phase 14: Cloud Architecture — Cheat Sheet

## Well-Architected (6 Pillars)
```
Ops Excellence | Security | Reliability
Performance    | Cost Opt | Sustainability
```

## Multi-AZ vs Multi-Region
```
Multi-AZ:     2-3 AZs, auto-failover, 1.5x cost (standard prod)
Multi-Region Active-Passive: DR, DNS failover, RPO=minutes
Multi-Region Active-Active:  Global, both serve, RPO≈0, 2x+ cost
```

## Serverless Decision
```
< 1M req/month:    Lambda (cheapest)
1M-100M req/month: Fargate / EKS
> 100M req/month:  EKS on EC2 (most economical)
Stateful/long:     EKS
Event-driven:      Lambda
```

## Cost Optimization
```
Compute:  Reserved (30-60% off), Spot (90% off, interruptible)
Storage:  Lifecycle policies, delete unused EBS, gp3 > gp2
Network:  VPC endpoints, CloudFront, compress responses
Monitor:  Cost Explorer + Kubecost + tag everything
```

## Cloud-Agnostic Stack
```
IaC: Terraform     Compute: K8s (EKS/GKE/AKS)
CI/CD: GitHub Actions  Monitoring: Prometheus+Grafana
Mesh: Istio        Secrets: Vault
```