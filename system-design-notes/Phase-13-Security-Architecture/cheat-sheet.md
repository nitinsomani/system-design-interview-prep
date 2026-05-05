# Phase 13: Security Architecture — Cheat Sheet

## Zero Trust
```
Never trust, always verify. Least privilege. Assume breach.
mTLS between services. NetworkPolicy deny-all default.
RBAC per namespace. Pod Security Standards: restricted.
```

## IAM Best Practices
```
No root account for daily use     MFA everywhere
Roles over users for services     Least privilege
IRSA: K8s SA → AWS IAM Role       SCPs for guardrails
IAM Access Analyzer               Short-lived credentials
```

## Secrets Management
```
Vault:            Dynamic secrets, audit logging, multi-cloud
Secrets Manager:  AWS-managed, auto rotation
Sealed Secrets:   Encrypted in Git, decrypt in cluster
SOPS:             Encrypt files with KMS
NEVER: secrets in Git, Dockerfiles, Slack, or long-lived creds
```

## Supply Chain Security
```
Base images:    Official, minimal (distroless)
Scan:           Trivy/Snyk in CI (block CRITICAL)
Sign:           cosign (verify provenance)
Admit:          OPA policy: approved registry + signed
SBOM:           Syft (generate) + Grype (scan)
Dependencies:   Dependabot/Renovate + lock files
```

## Encryption
```
Transit:  TLS 1.3, mTLS (service mesh), cert-manager
At rest:  S3 SSE-KMS, EBS encrypted, RDS encrypted, etcd encryption
KMS:      CMK + auto rotation + envelope encryption
```

## Network Security
```
WAF:              OWASP Top 10 protection (ALB/CloudFront)
Shield:           DDoS (Standard=free, Advanced=managed)
Security Groups:  Stateful, instance-level
NACLs:            Stateless, subnet-level
VPC Endpoints:    AWS services without internet
Flow Logs:        Network traffic audit
```