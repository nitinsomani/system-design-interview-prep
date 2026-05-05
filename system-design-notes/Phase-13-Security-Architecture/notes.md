# Phase 13: Security Architecture — Notes

## 1. Zero Trust Architecture

```
Principle: "Never trust, always verify"
  Traditional: Trust inside network perimeter
  Zero Trust: No implicit trust, verify every request

Pillars:
  1. Verify identity (AuthN): Who are you? (mTLS, JWT, OIDC)
  2. Verify authorization (AuthZ): What can you do? (RBAC, ABAC, OPA)
  3. Least privilege: Minimum permissions needed
  4. Micro-segmentation: Network policies between services
  5. Continuous verification: Re-validate throughout session
  6. Assume breach: Encrypt, log, monitor everything

Implementation in K8s:
  - Service mesh (Istio/Linkerd): mTLS between all services
  - NetworkPolicy: deny-all default, allow specific flows
  - RBAC: namespace-scoped roles
  - Pod Security Standards: restricted (no root, no host)
  - Audit logging: API server audit policy
```

## 2. Identity & Access Management (IAM)

```
AWS IAM:
  Users:    Human identities (avoid long-term keys)
  Roles:    Assumed by services/users (short-lived credentials)
  Policies: JSON documents defining permissions
  
  Best practices:
    - No root account for daily use
    - MFA on all human accounts
    - Use roles, not users, for services
    - Least privilege (start with zero, add as needed)
    - SCPs for organization-wide guardrails
    - IAM Access Analyzer to find overly permissive policies

IRSA (IAM Roles for Service Accounts):
  K8s ServiceAccount → AWS IAM Role
  Pod gets temporary AWS credentials (no static keys)
  
  Flow:
    Pod → K8s SA → OIDC → AWS STS → Temporary credentials
    Each service gets only the AWS permissions it needs
```

## 3. Secrets Management

```
HashiCorp Vault:
  Centralized secret store
  Dynamic secrets: generate DB credentials on-demand (TTL)
  Encryption as a service: transit engine
  Audit logging: every secret access logged

  K8s integration:
    - Vault Agent Injector: sidecar injects secrets as files
    - Vault CSI Provider: mount secrets as volumes
    - External Secrets Operator: sync to K8s Secret objects

AWS Secrets Manager:
  Managed service, automatic rotation
  Integration: Lambda rotation function
  Cost: $0.40/secret/month + $0.05/10K API calls

Secret hierarchy:
  1. Cloud-native (Secrets Manager): simple, managed
  2. Vault: complex, multi-cloud, dynamic secrets
  3. Sealed Secrets: encrypted in Git (GitOps)
  4. SOPS: encrypt files with KMS (GitOps)

NEVER:
  - Store secrets in Git (even private repos)
  - Use environment variables for secrets in Dockerfiles
  - Share secrets via Slack/email
  - Use long-lived credentials when short-lived available
```

## 4. Supply Chain Security

```
Container image security:
  1. Base image: use official, minimal (distroless)
  2. Scan: Trivy, Snyk (scan in CI, block critical CVEs)
  3. Sign: cosign/Notary (verify image provenance)
  4. Admit: only allow signed images from approved registries

  Admission control:
    OPA/Gatekeeper policy:
      - Images must be from ecr.amazonaws.com/*
      - Images must have no CRITICAL vulnerabilities
      - Images must be signed

Software Bill of Materials (SBOM):
  List of all components in your software
  Tools: Syft (generate), Grype (scan SBOM for vulns)
  Required by: US Executive Order on Cybersecurity

Dependency management:
  - Dependabot/Renovate: automated dependency updates
  - Lock files: pin exact versions
  - Private registry: mirror public packages (avoid supply chain attacks)
```

## 5. Encryption

```
In transit:
  TLS 1.3 everywhere (minimum TLS 1.2)
  mTLS between services (service mesh)
  Certificate management: cert-manager + Let's Encrypt

At rest:
  S3: SSE-S3 or SSE-KMS (default encryption)
  EBS: encrypted volumes (KMS)
  RDS: encrypted storage + encrypted snapshots
  K8s Secrets: enable etcd encryption at rest

KMS (Key Management Service):
  AWS-managed keys or customer-managed keys (CMK)
  Key rotation: automatic annual rotation
  Envelope encryption: data key encrypts data, KMS key encrypts data key
```

## 6. Network Security

```
WAF (Web Application Firewall):
  Protect against OWASP Top 10
  AWS WAF: rate limiting, SQL injection, XSS rules
  Place in front of ALB/CloudFront

DDoS Protection:
  AWS Shield Standard: free, automatic L3/L4 protection
  AWS Shield Advanced: L7 protection, DRT support

VPC Security:
  Security Groups: stateful, instance-level firewall
  NACLs: stateless, subnet-level firewall
  VPC Flow Logs: network traffic logging
  Private subnets: no direct internet access
  VPC Endpoints: access AWS services without internet
```

---

> **DevOps focus**: Vault operations, IRSA setup, image scanning in CI, cert-manager, network policies, audit logging.