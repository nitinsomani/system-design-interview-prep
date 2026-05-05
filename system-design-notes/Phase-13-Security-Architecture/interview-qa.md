# Phase 13: Security Architecture — Interview Q&A

## Q1: How do you implement zero trust in a Kubernetes environment?
**Answer:** Layer-by-layer: 1) **Network**: NetworkPolicy deny-all default → whitelist allowed flows. 2) **Identity**: service mesh (Istio) for automatic mTLS between all pods — every request is authenticated and encrypted. 3) **Authorization**: OPA/Gatekeeper policies + K8s RBAC (namespace-scoped roles). 4) **Pod Security**: restricted Pod Security Standards — no root, no host network, read-only root filesystem, drop all capabilities. 5) **Secrets**: External Secrets Operator syncing from Vault, short-lived credentials via IRSA. 6) **Audit**: API server audit logging to S3, all events tracked. 7) **Supply chain**: only signed images from approved registries admitted.

## Q2: How do you manage secrets across multiple environments?
**Answer:** Centralized with environment isolation: 1) **HashiCorp Vault** with separate secret engines per environment (secret/staging/*, secret/production/*). 2) K8s integration via **External Secrets Operator**: each namespace has ExternalSecret CRDs pointing to correct Vault path. 3) **IRSA/AppRole** per environment: staging pods can only access staging secrets. 4) **Dynamic secrets** for databases: Vault generates short-lived DB credentials per pod (auto-expires). 5) **Rotation**: Vault handles rotation, pods get new creds automatically. 6) For GitOps: **Sealed Secrets** — encrypted in Git, only cluster's controller can decrypt.

## Q3: How would you secure a CI/CD pipeline?
**Answer:** Multiple layers: 1) **Code**: branch protection, required reviews, signed commits. 2) **Dependencies**: Dependabot/Renovate for updates, lock files pinning versions. 3) **Build**: SAST (Semgrep), container image scan (Trivy — block CRITICAL/HIGH), SBOM generation. 4) **Artifact**: sign images with cosign, push to private registry only. 5) **Deploy**: OPA admission policy — only signed images from approved registry. 6) **Secrets**: CI secrets in GitHub/GitLab secrets store (not in code), rotate regularly. 7) **Access**: OIDC for CI → AWS (no static keys), least-privilege IAM roles per pipeline.

## Q4: Explain the principle of least privilege and how you enforce it.
**Answer:** Grant minimum permissions needed to perform a function. Enforcement: 1) **AWS IAM**: start with zero permissions, add specific actions for specific resources. Use IAM Access Analyzer to find unused permissions → tighten. 2) **K8s RBAC**: Role per namespace (not ClusterRole), bind to ServiceAccount (not user). 3) **Database**: application-specific DB users with only needed table/operation access. 4) **Network**: security groups allowing only required ports/sources. 5) **Pod**: drop all Linux capabilities, add only needed ones. 6) **Audit**: review permissions quarterly, revoke unused access.

## Q5: How do you handle a security incident (e.g., leaked credentials)?
**Answer:** Immediate: 1) **Revoke**: rotate/disable the compromised credentials immediately. 2) **Assess scope**: what resources were accessible with those creds? Check CloudTrail/audit logs for unauthorized access. 3) **Contain**: if compromise is active, isolate affected resources (security group changes, disable accounts). 4) **Remediate**: patch the vulnerability that led to the leak, rotate all potentially affected secrets. 5) **Communicate**: notify security team, affected stakeholders, possibly customers (depending on data accessed). 6) **Postmortem**: how did it happen, how to prevent recurrence. Action items: automated secret scanning in CI (git-secrets, truffleHog), pre-commit hooks.

## Rapid-Fire
- **mTLS?** → Both client and server present certificates, mutual authentication
- **OIDC?** → OpenID Connect: identity layer on OAuth 2.0, used for SSO
- **SCP?** → Service Control Policy: organization-level permission boundary
- **IRSA?** → IAM Roles for Service Accounts: K8s pods get AWS IAM roles
- **cosign?** → Sigstore tool to sign and verify container images
- **OPA vs Kyverno?** → OPA: Rego language, powerful. Kyverno: YAML policies, simpler