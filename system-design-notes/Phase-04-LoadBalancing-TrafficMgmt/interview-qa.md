# Phase 4: Load Balancing & Traffic Management — Interview Q&A

## Q1: L4 vs L7 load balancing — when to use each?
**Answer:** L4 (NLB): routes by IP+port, very fast, supports any TCP/UDP protocol (databases, custom protocols, gRPC passthrough). L7 (ALB): understands HTTP, routes by path/header/cookie, terminates TLS, supports canary deployments. Use L4 when you need raw performance or non-HTTP protocols. Use L7 when you need path-based routing, TLS termination, or HTTP-aware features. In K8s: Service = L4, Ingress = L7, Service Mesh = L7+.

## Q2: How would you reduce latency for global users?
**Answer:** 1) CDN for static assets at edge (CloudFront). 2) DNS latency-based routing to nearest region (Route 53). 3) Edge computing for dynamic content (CloudFront Functions/Lambda@Edge). 4) Multi-region deployment with data replication. 5) Connection keep-alive + HTTP/2 multiplexing. 6) Compress responses (gzip/brotli). Each layer shaves off latency — CDN alone reduces P50 from 200ms to 20ms for cached content.

## Q3: How do you handle CDN cache invalidation?
**Answer:** Best approach: content-hashed filenames (style.abc123.css) with long Cache-Control (1 year) — new deploy generates new hash, old files expire naturally. For APIs: short s-maxage (30-60s) or stale-while-revalidate. Path invalidation (CloudFront invalidation API) is a fallback — takes 5-15 min. Never use query strings for cache busting (CDN may ignore them).

## Q4: Explain consistent hashing and its use in load balancing.
**Answer:** Regular hash: add/remove server → all keys remap. Consistent hash: hash ring where only keys between the changed server and next server remap (~1/N keys move). Used in: Redis Cluster (slot mapping), CDN routing, session affinity. Virtual nodes: each server gets multiple points on ring for even distribution. Benefits: minimal disruption during scaling events.

## Q5: How does rate limiting work at scale?
**Answer:** Multi-level: WAF (per-IP at edge), API Gateway (per-user via Redis counter), service mesh (per-service via Envoy). Algorithm: token bucket for burst tolerance. Implementation: shared Redis counter across instances — `INCR rate:user123:minute` with TTL. Distributed rate limiting challenge: each instance has local counter → use centralized Redis or accept approximate limiting with local counters + periodic sync.

## Rapid-Fire
- **Sticky sessions downside?** → Uneven load, failover loses session
- **Anycast?** → Same IP from multiple locations, BGP routes to nearest
- **CDN origin?** → Backend server CDN fetches from on cache miss
- **301 vs 302?** → 301: permanent (cached), 302: temporary (not cached)
- **TLS termination?** → Decrypt HTTPS at LB, forward HTTP to backends
- **gRPC LB challenge?** → HTTP/2 persistent connection → L7 per-stream routing needed