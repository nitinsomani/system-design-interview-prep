# Phase 10: CI/CD & Deployments — Lab Exercises

## Lab 1: ArgoCD Setup & GitOps Workflow
```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Create application
cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/k8s-manifests.git
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF

# Verify sync status
argocd app get my-app
```

## Lab 2: Argo Rollouts Canary
```bash
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Create canary rollout
cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata: { name: web-app }
spec:
  replicas: 5
  selector: { matchLabels: { app: web-app } }
  template:
    metadata: { labels: { app: web-app } }
    spec:
      containers:
        - name: app
          image: nginx:1.24
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: { duration: 30s }
        - setWeight: 50
        - pause: { duration: 30s }
        - setWeight: 100
EOF

# Trigger canary by updating image
kubectl argo rollouts set image web-app app=nginx:1.25

# Watch progress
kubectl argo rollouts get rollout web-app --watch

# Abort if needed
kubectl argo rollouts abort web-app
```

## Lab 3: GitHub Actions CI Pipeline
```yaml
# .github/workflows/ci.yml
name: CI
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lint
        run: make lint

      - name: Unit Tests
        run: make test

      - name: Build Image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Security Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          severity: HIGH,CRITICAL
          exit-code: 1

      - name: Push to ECR
        if: github.ref == 'refs/heads/main'
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPO
          docker tag myapp:${{ github.sha }} $ECR_REPO:${{ github.sha }}
          docker push $ECR_REPO:${{ github.sha }}

      - name: Update Manifests
        if: github.ref == 'refs/heads/main'
        run: |
          cd k8s-manifests
          kustomize edit set image myapp=$ECR_REPO:${{ github.sha }}
          git commit -am "Update image to ${{ github.sha }}"
          git push
```

## Lab 4: Deployment Strategy Comparison Exercise
```
For each service, choose the best deployment strategy:

1. Payment service (critical, 1000 TPS)
   → Canary (10% → monitor → 100%)
   Reason: Minimize blast radius for revenue-critical service

2. Internal admin dashboard (10 users)
   → Recreate
   Reason: Low traffic, simplest, brief downtime acceptable

3. API gateway (all traffic flows through)
   → Blue-green
   Reason: Instant rollback needed, can't afford degraded state

4. Background worker (processes queue)
   → Rolling update
   Reason: No user-facing traffic, graceful shutdown + new pods

5. ML model serving (needs A/B comparison)
   → Canary with traffic mirror
   Reason: Compare model versions on real traffic without risk
```