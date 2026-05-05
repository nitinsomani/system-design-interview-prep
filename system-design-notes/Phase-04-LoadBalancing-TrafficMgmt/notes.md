# Phase 4: Load Balancing & Traffic Management — Notes

## 1. Load Balancing Fundamentals

```
L4 (Transport Layer):
  Routes by IP + port, no payload inspection
  Fast, supports any TCP/UDP protocol
  Examples: AWS NLB, HAProxy TCP, K8s Service (iptables)

L7 (Application Layer):
  Routes by HTTP headers, URL path, cookies
  Can modify requests, terminate TLS, compress
  Examples: AWS ALB, Nginx, Traefik, Envoy, HAProxy HTTP

Algorithms:
  Round Robin:       Equal distribution (default)
  Weighted RR:       More traffic to stronger backends
  Least Connections: Route to least busy backend
  IP Hash:           Same client → same backend (sticky)
  Consistent Hash:   Minimal disruption when backends change
  Random:            Simple, works well at scale
```

## 2. Global Server Load Balancing (GSLB)

```
Route users to nearest region:

  DNS-based GSLB:
    Route 53 latency-based routing
    User in EU → eu-west-1, User in US → us-east-1
    Pros: Simple, no extra infra
    Cons: DNS TTL caching delays failover

  Anycast:
    Same IP announced from multiple locations via BGP
    Network routes to nearest location
    Pros: Instant failover, no DNS dependency
    Cons: Complex BGP management
    Used by: Cloudflare, Google Cloud LB

  CDN as GSLB:
    CloudFront/Akamai: edge locations worldwide
    Static: serve from cache at edge
    Dynamic: route to nearest origin
```

## 3. CDN (Content Delivery Network)

```
CDN caches content at edge locations close to users

What to cache:
  Static: CSS, JS, images, fonts, videos
  Dynamic: API responses (with short TTL)
  Full pages: Server-side rendered pages

Cache-Control headers:
  Cache-Control: public, max-age=31536000  (1 year, static assets)
  Cache-Control: private, max-age=0        (never cache, user-specific)
  Cache-Control: public, s-maxage=60       (CDN caches for 60s)

CDN invalidation:
  Path invalidation: CloudFront → Invalidate /images/*
  Versioned URLs: /style.v2.css (never need invalidation)
  
  Key insight: Use versioned file names (content hash)
    styles.a1b2c3.css → cache forever, new deploy = new hash
```

## 4. Rate Limiting & Throttling

```
Levels:
  Edge/CDN:     Per-IP rate limiting (WAF rules)
  API Gateway:  Per-user/API-key limiting
  Service Mesh: Per-service limiting
  Application:  Per-endpoint limiting

Algorithms:
  Token Bucket:    Allows bursts up to bucket size
  Sliding Window:  Count requests in moving time window
  Leaky Bucket:    Constant output rate, queue excess

Response: 429 Too Many Requests
  Retry-After: 30
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 0
```

## 5. Traffic Shaping for Deployments

```
Canary:          10% → 50% → 100% (gradual)
Blue-Green:      Switch 100% at once (instant rollback)
Traffic Mirror:  Copy to new version (zero risk testing)
A/B Testing:     Route by user attribute (cookie, header)
Feature Flags:   Enable feature per user/group (LaunchDarkly)

Istio VirtualService:
  route:
    - destination: { host: app, subset: v1 }
      weight: 90
    - destination: { host: app, subset: v2 }
      weight: 10
```

---

> **DevOps focus**: LB choice (L4 vs L7), CDN caching strategy, and traffic management for safe deployments.