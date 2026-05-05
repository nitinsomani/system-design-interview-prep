# Phase 10: CI/CD & Deployment Strategies — Notes

## 1. CI/CD Pipeline Architecture

```
Continuous Integration:
  Code → Commit → Build → Test → Artifact
  
  Stages:
    1. Lint + Static Analysis (seconds)
    2. Unit Tests (minutes)
    3. Build Container Image (minutes)
    4. Push to Registry (seconds)
    5. Security Scan (SAST, image scan) (minutes)

Continuous Delivery:
  Artifact → Deploy to Staging → Integration Tests → Manual Approval → Production

Continuous Deployment:
  Artifact → Deploy to Staging → Tests → Auto Deploy to Production

Tools:
  CI: GitHub Actions, GitLab CI, Jenkins, CircleCI
  CD: ArgoCD, Flux, Spinnaker
  Registry: ECR, GCR, Docker Hub, Harbor
```

## 2. Deployment Strategies

```
Rolling Update:
  Replace pods gradually (default K8s strategy)
  maxSurge: 25%       (extra pods during update)
  maxUnavailable: 25%  (pods that can be down)
  Pros: Zero downtime, gradual rollout
  Cons: Two versions running simultaneously, slow rollback
  Rollback: kubectl rollout undo deploy/app

Blue-Green:
  Two identical environments (blue = current, green = new)
  Deploy to green → test → switch traffic → done
  Rollback: switch back to blue (instant)
  Pros: Instant rollback, full testing before traffic
  Cons: 2x infrastructure cost during deployment
  Implementation: K8s Service selector switch or Ingress weight

Canary:
  Route small % of traffic to new version
  10% → monitor → 50% → monitor → 100%
  Pros: Minimal blast radius, real traffic testing
  Cons: Slower rollout, complex traffic splitting
  Tools: Istio, Argo Rollouts, Flagger

Recreate:
  Kill all old → deploy new (downtime)
  Only for: dev environments, stateful apps that can't run two versions
```

## 3. GitOps

```
Principle: Git is the single source of truth for infrastructure and apps

Push-based (traditional):
  CI pipeline pushes changes to cluster
  CI needs cluster credentials (security concern)

Pull-based (GitOps):
  Agent in cluster pulls desired state from Git
  No external access to cluster needed
  Tools: ArgoCD, Flux

ArgoCD workflow:
  1. Developer commits code → CI builds image → pushes to ECR
  2. CI updates image tag in Git manifest repo
  3. ArgoCD detects drift between Git and cluster
  4. ArgoCD syncs: applies manifests to cluster
  5. ArgoCD reports sync status (healthy/degraded/progressing)

Repository structure:
  app-code-repo/        (source code, Dockerfile, CI pipeline)
  k8s-manifests-repo/   (Helm charts or Kustomize overlays)
    ├── base/
    │   ├── deployment.yaml
    │   └── service.yaml
    ├── overlays/
    │   ├── staging/
    │   └── production/
```

## 4. Progressive Delivery with Argo Rollouts

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: app
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        - setWeight: 10      # 10% traffic to canary
        - pause: { duration: 5m }
        - setWeight: 30
        - pause: { duration: 5m }
        - setWeight: 60
        - pause: { duration: 5m }
        - setWeight: 100
      analysis:
        templates:
          - templateName: success-rate
        startingStep: 1
        args:
          - name: service-name
            value: app
```

## 5. Feature Flags

```
Decouple deployment from release:
  Deploy: code goes to production (dark launch)
  Release: feature enabled for users (toggle on)

Levels:
  Boolean:    ON/OFF per feature
  Percentage: 10% of users see new feature
  User-based: Enable for beta users, disable for others
  Env-based:  ON in staging, OFF in production

Tools: LaunchDarkly, Unleash, Flagsmith, ConfigCat

Benefits:
  - Kill switch: disable broken feature without deploy
  - A/B testing: measure impact of changes
  - Gradual rollout: independent of deployment
  - Trunk-based development: merge incomplete features safely

Cleanup: Remove old flags after feature is 100% rolled out
  Tech debt: unused flags accumulate → track and clean regularly
```

## 6. Rollback Strategies

```
Code rollback:
  kubectl rollout undo deploy/app
  ArgoCD: revert Git commit → auto-sync

Database rollback:
  Forward-only migrations (expand-contract)
  Never rollback schema changes — add new, deprecate old
  Data rollback: restore from backup (last resort)

Config rollback:
  Git-tracked configs → revert commit
  Feature flag: toggle OFF

Automated rollback:
  Argo Rollouts: analysis template fails → auto rollback
  Flagger: canary error rate > threshold → abort + rollback
```

---

> **DevOps focus**: GitOps (ArgoCD), progressive delivery, rollback automation, pipeline optimization, feature flag lifecycle.