# Phase 4: Load Balancing & Traffic Management — Cheat Sheet

## LB Types
```
L4: IP+port, fast, any protocol (NLB, HAProxy TCP, kube-proxy)
L7: HTTP-aware, path/header routing, TLS termination (ALB, Nginx, Envoy)
```

## Algorithms
```
Round Robin      Equal rotation       Least Conn    Fewest active
Weighted RR      Proportional         IP Hash       Session sticky
Consistent Hash  Minimal disruption   Random        Simple, effective
```

## GSLB Options
```
DNS-based:  Route 53 latency/geo routing (TTL-dependent failover)
Anycast:    Same IP from multiple locations via BGP (instant)
CDN:        CloudFront/Akamai edge routing
```

## CDN Cache-Control
```
Static (1yr):    Cache-Control: public, max-age=31536000
No cache:        Cache-Control: private, no-store
CDN only (60s):  Cache-Control: public, s-maxage=60
Best practice:   Versioned filenames (style.abc123.css) + long TTL
```

## Rate Limiting
```
Token Bucket:    Allows bursts         429 Too Many Requests
Sliding Window:  Smooth limiting       Retry-After: 30
Leaky Bucket:    Constant output rate  X-RateLimit-Remaining: 0
```

## Traffic Management
```
Canary:     Gradual % shift (10→50→100)
Blue-Green: Instant 100% switch + rollback
Mirror:     Shadow traffic (zero risk)
Feature Flag: Per-user toggle (LaunchDarkly)
```