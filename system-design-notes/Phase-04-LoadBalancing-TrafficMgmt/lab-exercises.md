# Phase 4: Load Balancing & Traffic Management — Lab Exercises

## Lab 1: K8s Ingress with Path-Based Routing
```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: app.local
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service: { name: api-svc, port: { number: 80 } }
          - path: /
            pathType: Prefix
            backend:
              service: { name: frontend-svc, port: { number: 80 } }
EOF
# Test: curl -H "Host: app.local" http://<ingress-ip>/api → api-svc
# Test: curl -H "Host: app.local" http://<ingress-ip>/ → frontend-svc
```

## Lab 2: Rate Limiting with Nginx Ingress
```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-limited
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "10"
    nginx.ingress.kubernetes.io/limit-burst-multiplier: "5"
spec:
  ingressClassName: nginx
  rules:
    - host: api.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: { name: api-svc, port: { number: 80 } }
EOF
# Test: Send 20 rapid requests → observe 429 responses after 10
```

## Lab 3: CDN Cache-Control Headers
```bash
# Nginx config for proper cache headers
# Static assets: long cache with versioned names
location ~* \.(css|js|png|jpg|gif|ico|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
# API responses: short CDN cache
location /api/ {
    add_header Cache-Control "public, s-maxage=30, max-age=0";
}
# User-specific: never cache
location /api/me {
    add_header Cache-Control "private, no-store";
}
```

## Lab 4: Traffic Split Design Exercise
```
Exercise: Design canary deployment for payment-service
  Step 1: Deploy v2 alongside v1
  Step 2: VirtualService: 95% → v1, 5% → v2
  Step 3: Monitor v2: error rate, P99 latency, business metrics
  Step 4: If healthy after 30 min → shift to 50/50
  Step 5: If healthy after 1 hour → shift to 100% v2
  Step 6: Remove v1 deployment
  Rollback: At any step, shift 100% back to v1
```