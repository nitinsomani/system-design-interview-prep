# Phase 10: CI/CD & Deployments — Interview Q&A

## Q1: How do you design a CI/CD pipeline for a microservices platform?
**Answer:** Per-service pipeline: 1) **CI** (on PR): lint, unit tests, build image with PR SHA tag, SAST scan, push to staging registry. 2) **On merge to main**: build final image, tag with Git SHA, push to ECR, update manifest repo with new image tag. 3) **CD** (ArgoCD): detects manifest change → syncs to staging → run integration/smoke tests → manual approval → sync to production. Key decisions: monorepo vs polyrepo (polyrepo per service for independent pipelines), shared CI templates for consistency, artifact promotion (same image across environments, different configs).

## Q2: Compare blue-green, canary, and rolling deployments.
**Answer:** **Rolling** (default K8s): gradual pod replacement, zero downtime, but two versions coexist temporarily. Good for most services. **Blue-green**: full parallel environment, instant switch, instant rollback. Good for critical services where you need full testing before any user traffic. Cost: 2x infra during deploy. **Canary**: route small % to new version, monitor, gradually increase. Best for high-traffic services where you need real-user validation. Requires traffic splitting (Istio/Argo Rollouts). My preference: canary with automated analysis for critical services, rolling for everything else.

## Q3: What is GitOps and why does it matter?
**Answer:** GitOps: Git is the single source of truth for both application and infrastructure state. An agent (ArgoCD/Flux) running in-cluster continuously reconciles actual state with desired state in Git. Benefits: 1) Audit trail — every change is a Git commit. 2) Security — CI doesn't need cluster credentials. 3) Disaster recovery — recreate cluster by pointing ArgoCD to Git. 4) Consistency — drift detection and auto-correction. 5) Developer experience — PR-based workflow for infra changes. We separate app code repo from manifest repo so image builds don't trigger unnecessary syncs.

## Q4: How do you handle database schema changes in CI/CD?
**Answer:** Expand-contract pattern with forward-only migrations: 1) **Expand**: add new column (nullable, no constraints). Deploy app that writes to both old and new columns. 2) **Migrate**: backfill data. 3) **Contract**: deploy app that reads only from new column. Add constraints. Drop old column in next release. Tools: Flyway/Liquibase embedded in app startup, or separate migration job (K8s Job). Rules: never rename/drop in one step, never add NOT NULL without default, test on production-copy data, always have rollback SQL (even if you don't use it).

## Q5: How do you implement automated rollback?
**Answer:** Multiple layers: 1) **Argo Rollouts analysis**: define AnalysisTemplate with success criteria (error rate <1%, P99 <200ms). If analysis fails during canary → automatic rollback. 2) **K8s readiness/liveness probes**: pods that don't pass health checks don't receive traffic and get restarted. 3) **Pipeline gates**: smoke tests in staging — fail → block production deploy. 4) **Feature flags**: if issue detected post-deploy, toggle flag OFF (no code deploy needed). 5) **Manual**: `kubectl rollout undo` or revert Git commit (ArgoCD auto-syncs). Key: the faster you can detect and rollback, the smaller the blast radius.

## Rapid-Fire
- **Image tag: latest vs SHA?** → Always SHA/semver. "latest" is not reproducible.
- **Monorepo vs polyrepo CI?** → Polyrepo: independent pipelines. Monorepo: path-based triggers.
- **ArgoCD sync waves?** → Deploy resources in order: namespace → secrets → deployment.
- **Trunk-based development?** → All developers commit to main, use feature flags for incomplete work.
- **Pipeline caching?** → Cache dependencies (npm, pip) and Docker layers to speed up builds.
- **DORA metrics?** → Deployment frequency, lead time, change failure rate, MTTR.