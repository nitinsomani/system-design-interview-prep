# Phase 10: CI/CD & Deployments — Cheat Sheet

## CI Pipeline Stages
```
Lint → Unit Test → Build Image → Push Registry → Security Scan
Fast feedback: fail early (lint first, slow tests last)
```

## Deployment Strategies
```
Rolling:     Gradual replace, zero downtime, slow rollback
Blue-Green:  Switch traffic instantly, 2x cost, instant rollback
Canary:      10→50→100% gradual, minimal blast radius
Recreate:    All down → all up (downtime, dev only)
```

## GitOps (ArgoCD)
```
Git = source of truth → Agent pulls from Git → syncs to cluster
Repo structure: app-repo (code+CI) + manifests-repo (K8s YAML)
Benefits: audit trail, declarative, no cluster creds in CI
```

## Argo Rollouts Canary
```
setWeight: 10 → pause 5m → 30 → pause → 60 → pause → 100
+ analysis template: auto-rollback if error rate > threshold
```

## Feature Flags
```
Deploy ≠ Release
Kill switch: disable feature without deploy
Cleanup: remove old flags to avoid tech debt
Tools: LaunchDarkly, Unleash, Flagsmith
```

## Rollback
```
Code:    kubectl rollout undo / revert Git commit
DB:      Forward-only migrations (expand-contract)
Config:  Revert Git commit / toggle feature flag
Auto:    Argo Rollouts analysis template → auto abort
```