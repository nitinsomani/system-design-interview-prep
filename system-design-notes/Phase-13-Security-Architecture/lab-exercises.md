# Phase 13: Security Architecture — Lab Exercises

## Lab 1: K8s NetworkPolicy (Zero Trust)
```bash
cat <<EOF | kubectl apply -f -
# Deny all traffic by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: deny-all, namespace: production }
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
# Allow frontend → backend only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: allow-frontend-to-backend, namespace: production }
spec:
  podSelector: { matchLabels: { app: backend } }
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector: { matchLabels: { app: frontend } }
      ports:
        - { port: 8080, protocol: TCP }
---
# Allow backend → database only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: allow-backend-to-db, namespace: production }
spec:
  podSelector: { matchLabels: { app: database } }
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector: { matchLabels: { app: backend } }
      ports:
        - { port: 5432, protocol: TCP }
EOF
# Test: frontend → backend (allowed), frontend → database (blocked)
```

## Lab 2: External Secrets Operator
```bash
# Install ESO
helm install external-secrets external-secrets/external-secrets \
  -n external-secrets --create-namespace

# Create SecretStore (AWS Secrets Manager)
cat <<EOF | kubectl apply -f -
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata: { name: aws-secrets, namespace: production }
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef: { name: external-secrets-sa }
---
# Sync a secret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata: { name: db-creds, namespace: production }
spec:
  refreshInterval: 1h
  secretStoreRef: { name: aws-secrets, kind: SecretStore }
  target: { name: db-credentials }
  data:
    - secretKey: DB_PASSWORD
      remoteRef: { key: production/db-password }
EOF
# Result: K8s Secret "db-credentials" auto-synced from AWS
```

## Lab 3: Image Scanning in CI
```bash
# Scan with Trivy
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:latest

# Output example:
# Total: 3 (HIGH: 2, CRITICAL: 1)
# ┌──────────┬──────────────┬──────────┬─────────┐
# │ Library  │ Vulnerability│ Severity │ Fix     │
# ├──────────┼──────────────┼──────────┼─────────┤
# │ openssl  │ CVE-2024-XXX │ CRITICAL │ 3.1.5   │
# └──────────┴──────────────┴──────────┴─────────┘
# Exit code 1 → pipeline fails → fix before merge

# Generate SBOM
syft myapp:latest -o spdx-json > sbom.json

# Scan SBOM for vulnerabilities
grype sbom:sbom.json
```

## Lab 4: Security Architecture Design Exercise
```
Scenario: Design security for a fintech platform

  Layers:
  1. Edge: WAF (AWS WAF) + Shield Advanced + CloudFront
  2. Network: Private subnets, VPC endpoints, no public IPs
     NACLs + Security Groups (least privilege ports)
  3. Identity: OIDC (Okta) → K8s RBAC, IRSA for AWS access
  4. Service-to-service: Istio mTLS (automatic, all traffic)
  5. Secrets: Vault with dynamic DB credentials (1hr TTL)
     No static secrets anywhere
  6. Data: KMS encryption at rest (S3, EBS, RDS)
     TLS 1.3 in transit, field-level encryption for PII
  7. Supply chain: Trivy scan (zero CRITICAL), cosign signatures
     OPA: only signed images from private ECR
  8. Audit: CloudTrail, VPC Flow Logs, K8s audit logs → S3
     SIEM alerting on anomalies
  9. Compliance: SOC2 controls mapped to infra policies
     Automated compliance checks (AWS Config rules)
```