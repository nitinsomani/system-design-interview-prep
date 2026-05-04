# System Design Core Concepts & Notes

## Fundamentals

### CAP Theorem (Consistency, Availability, Partition Tolerance)
**Gilhoo's Theorem**: In a distributed system, you can only achieve **2 out of 3** guarantees simultaneously.

- **Consistency**: All nodes see the same data simultaneously (Strong consistency)
- **Availability**: System remains operational despite node failures 100% of the time
- **Partition Tolerance**: System continues to operate despite network partitions

**Practical Trade-offs:**
- **CA Systems** (Consistency + Availability): RDBMS in single data center (no network partitions)
- **CP Systems** (Consistency + Partition Tolerance): MongoDB, Redis, ZooKeeper (sacrifice availability)
- **AP Systems** (Availability + Partition Tolerance): Cassandra, DynamoDB, CouchDB (sacrifice consistency)

### ACID vs BASE

**ACID (Traditional RDBMS):**
- **Atomicity**: All operations succeed or fail as a unit
- **Consistency**: Database in valid state after transaction
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed changes persist despite failures

**BASE (NoSQL):**
- **Basically Available**: System guarantees availability (degraded service ok)
- **Soft State**: State can change over time (even without input)
- **Eventually Consistent**: System will become consistent over time

### Distributed System Challenges

**1. Network Reliability:**
- Networks are inherently unreliable (latency, packet loss, partitions)
- Design systems to handle failures gracefully

**2. Clock Synchronization:**
- Logical vs Physical clocks
- NTP synchronization limitations
- Use Lamport timestamps for ordering

**3. Consensus Problems:**
- Achieving agreement in distributed systems
- Paxos, Raft algorithms for leader election
- Two-phase commit (2PC), Three-phase commit (3PC)

**4. Failure Patterns:**
- Fail-stop vs Byzantine faults
- Partial failures: Some components fail while others work
- Cascading failures: Load from failed component overwhelms others

## Scalability Patterns

### Horizontal vs Vertical Scaling
**Vertical Scaling (Scale Up):**
- Increase individual server capacity (CPU, RAM, storage)
- **Pros**: Simple, no code changes needed
- **Cons**: Hardware limits, single point of failure, expensive

**Horizontal Scaling (Scale Out):**
- Add more identical servers
- **Pros**: High scalability, fault tolerance, cost-effective
- **Cons**: Complexity in data consistency, load distribution

### Load Balancing Strategies
**1. Round Robin:**
- Sequential distribution of requests
- Simple, no state required

**2. Least Connections:**
- Route to server with fewest active connections
- Better for long-running connections

**3. IP Hash:**
- Hash client IP for consistent server routing
- Useful for session persistence

**4. Weighted Load Balancing:**
- Assign weights based on server capacity
- Distribute load proportionally

**5. Layer 7 (Application-aware):**
- Route based on HTTP headers, cookies, content
- URL-based routing, content-based routing

### Database Scaling Patterns

**1. Read Replicas:**
- Multiple read-only copies of master database
- Horizontal read scaling
- Eventual consistency considerations

**2. Sharding (Horizontal Partitioning):**
- Split data across multiple databases
- Key-based, range-based, or hash-based sharding
- Resharding challenges with data growth

**3. Vertical Partitioning:**
- Split table into multiple tables by column
- Normalize schema for better performance

**4. Federation:**
- Split databases by function (Users DB, Products DB)
- Easier to scale teams independently

### Caching Strategies

**Cache-aside (Lazy Loading):**
- Application checks cache first, then database
- Cache fills on cache miss
- Pros: Cache stampede prevention, simple
- Cons: Cache misses hurt performance

```
Read: Cache → DB → Update Cache
Write: Update DB → Invalidate Cache
```

**Write-through:**
- Write to cache and database simultaneously
- Always read from cache (fast path)
- Pros: Data always fresh
- Cons: Write operations slower

**Write-behind (Write-back):**
- Write to cache, database asynchronously
- Performance improvement but data loss risk
- Requires careful failure handling

**Cache Invalidation:**
- **TTL (Time To Live)**: Expire entries after time
- **LRU (Least Recently Used)**: Remove least accessed items
- **Explicit Invalidation**: Application removes stale entries

### CDN (Content Delivery Network)
**Purpose:** Reduce latency by serving content from geographically distributed servers

**Benefits:**
- Lower latency for global users
- Reduce origin server load
- DDoS protection
- Cost savings on bandwidth

**Architecture:**
- **Edge Servers**: Global points of presence (PoP)
- **Origin Server**: Central content source
- **Caching Rules**: TTL, cache control headers
- **Dynamic Content**: API responses may not be cacheable

**Challenges:**
- Cache invalidation consistency
- Cost management for data transfer
- Debugging edge-specific issues

## Data Consistency Models

### Strong Consistency
- Reads always return most recent write
- Linearizability: Operations appear instantaneous
- **Trade-off:** Higher latency, lower availability
- **Examples:** RDBMS transactions, etcd, ZooKeeper

### Eventual Consistency
- Eventual convergence to consistent state
- No guarantees on operation ordering
- **Advantages:** High availability, low latency
- **Conflict Resolution:** Last-write wins, application logic

### Monotonic Reads
- Once a user reads a value, future reads return same or newer values
- No "going back in time"
- **Implementation:** Version stamps, read repairs

### Read Your Own Writes (RYOW)
- After user writes, they immediately see their own changes
- Stronger than eventual consistency
- **Use Case:** User profile updates

### Session Consistency
- Consistency guarantee within a single user session
- Multiple sessions can have different views
- **Implementation:** Sticky sessions, session store

## Microservices Architecture

### Decomposition Patterns
**1. Business Capability:**
- Split by business domains (User Management, Payment, Inventory)
- Align with organizational structure
- Independent deployment and scaling

**2. Subdomain:**
- Align with Domain-Driven Design bounded contexts
- Business logic encapsulation

**3. Strangler Fig:**
- Gradually migrate from monolith
- Start with leaf services, move inward
- Low-risk migration approach

### Service Communication

**Synchronous:**
- **REST APIs**: Request-response pattern
- **GraphQL APIs**: Flexible query language
- **gRPC**: High-performance RPC framework
- **Cons:** Coupling increases, slower responses

**Asynchronous:**
- **Message Queues**: Kafka, RabbitMQ, SQS
- **Event Streaming**: Publish-subscribe pattern
- **Pros:** Loose coupling, better scalability
- **Cons:** Debugging complexity, eventual consistency

### Data Management
**Database per Service:**
- Each service owns its data
- No cross-service foreign keys
- Data consistency through events=SAGA pattern
- Challenges: Cross-service queries, transactions

**CQRS (Command Query Responsibility Segregation):**
- Separate read and write models
- Write: Domain commands, validation logic
- Read: Optimized for query performance
- Event sourcing for audit trails

### Cross-Cutting Concerns
**Service Discovery:**
- Consul, etcd, Kubernetes DNS
- Dynamic service registration/discovery

**API Gateway:**
- Single entry point for client requests
- Authentication, rate limiting, routing
- Protocol translation (REST ↔ gRPC)

**Service Mesh:**
- Service-to-service communication layer
- Istio, Linkerd: Traffic management, security
- Circuit breaking, observability built-in

### Reliability Patterns

**Circuit Breaker:**
- Prevent cascade failures in dependent services
- States: Closed (normal), Open (fail fast), Half-Open (test)
- Hystrix, Resilience4j implementations

**Bulkhead:**
- Isolate failures within service boundaries
- Separate thread/connection pools
- Prevent one service from exhausting resources

**Retry with Exponential Backoff:**
- Intelligent retry with increasing delays
- Jitter to prevent thundering herd
- Maximum retry limit, circuit breaker integration

**Timeout:**
- Define maximum wait time for operations
- Network calls, database queries, API responses
- Fail fast to free up resources

## Real-time Systems

### WebSocket vs Server-Sent Events (SSE)
**WebSocket:**
- Bidirectional communication
- Binary and text support
- Better for interactive applications

**SSE:**
- Server to client only (unidirectional)
- Automatic reconnection
- Better for news feeds, notifications

### Push vs Pull Architectures
**Push Architecture:**
- Server broadcasts updates to clients
- Real-time notifications, live updates
- Examples: WebSocket, SSE, WebRTC

**Pull Architecture:**
- Clients poll for updates periodically
- AJAX long polling, polling
- Simpler, but less efficient

### Event-Driven Architecture
**Components:**
- **Event Producer**: Generate events
- **Event Broker**: Route and buffer events
- **Event Consumer**: Process events asynchronously

**Patterns:**
- **Event Sourcing**: Store state as sequence of events
- **CQRS**: Separate read/write models
- **Saga Pattern**: Long-running business transactions

## Security in System Design

### Authentication & Authorization
**Authentication Methods:**
- **JWT (JSON Web Tokens)**: Stateless, scalable
- **OAuth 2.0**: Delegated authorization
- **SAML**: Enterprise authentication

**Authorization Patterns:**
- **RBAC (Role-Based)**: Permissions based on user roles
- **ABAC (Attribute-Based)**: Fine-grained permission control
- **Zero Trust**: Never trust, always verify

### Data Protection
**Encryption at Rest:**
- KMS (Key Management Service)
- Database encryption, file system encryption

**Encryption in Transit:**
- TLS 1.3 for transport security
- Certificate management and rotation

**Secrets Management:**
- HashiCorp Vault, AWS Secrets Manager
- Dynamic credentials, lease-based access

### API Security
**Rate Limiting:**
- Token bucket, leaky bucket algorithms
- Distributed rate limiting with Redis

**Input Validation:**
- Schema validation, sanitization
- Prevent injection attacks (SQL, XSS)

**API Gateway Security:**
- Authentication middleware
- JWT validation, API key management
- Request throttling, abuse detection

## Performance Optimization

### Database Optimization
**Indexing Strategies:**
- Primary keys, composite indexes
- Partial indexes for subset queries
- Avoid over-indexing (write performance impact)

**Query Optimization:**
- Explain plan analysis
- Avoid SELECT *, limit result sets
- Batch operations vs individual queries

**Connection Management:**
- Connection pooling (HikariCP, pgBouncer)
- Connection multiplexing
- Prepared statements for repeated queries

### Application Performance
**Memory Management:**
- Garbage collection tuning
- Memory leaks detection
- Object pooling for expensive objects

**Asynchronous Processing:**
- Background job queues (Sidekiq, Celery)
- Reactive programming patterns
- Non-blocking I/O operations

**Caching Layers:**
- Multi-level caching (L1 CPU, L2 application, L3 distributed)
- Cache warming strategies
- Cache serialization formats

### Profiling & Monitoring
**APM Tools:**
- New Relic, DataDog, AppDynamics
- Flame graphs for CPU profiling
- Memory heap analysis

**Performance Testing:**
- Load testing (JMeter, k6, Artillery)
- Stress testing (Gradual load increases)
- Soak testing (Extended duration testing)

## Cloud Architecture

### Multi-Region vs Multi-Cloud
**Multi-Region:**
- Disaster recovery, low latency
- Cross-region data replication
- Geo-DNS routing (Route53, Cloudflare)

**Multi-Cloud:**
- Vendor lock-in avoidance
- Best-of-breed service selection
- Risk mitigation (vendor outages)

### Serverless Architecture
**FaaS Benefits:**
- Auto-scaling based on demand
- Reduced operational overhead
- Pay-per-request pricing

**Challenges:**
- Cold start latency
- Vendor lock-in (runtime, services)
- Stateless by design

**Patterns:**
- API Gateway + Lambda functions
- Event-driven processing (S3 triggers, EventBridge)
- Step Functions for orchestration

### Cost Optimization
**Compute Savings:**
- Reserved Instances (1-3 year commitments)
- Spot Instances (up to 90% discount)
- Rightsizing based on utilization

**Storage Savings:**
- S3 storage classes (Intelligent Tiering)
- Glacier for archival data
- Lifecycle policies for automatic tiering

**Network Savings:**
- Regional data transfer vs cross-region
- CDN for global distributions
- Private networking (VPC peering, Transit Gateway)

## Disaster Recovery

### RTO/RPO Definitions
**RTO (Recovery Time Objective):**
- Maximum acceptable downtime
- Time to restore business operations

**RPO (Recovery Point Objective):**
- Maximum data loss tolerance
- Time between last backup and failure

### DR Strategies
**1. Backup & Restore:**
- Simple, cost-effective
- Long recovery time
- Suitable for non-critical systems

**2. Pilot Light:**
- Core infrastructure running
- Scale up during disaster
- Faster recovery than backup/restore

**3. Warm Standby:**
- Scaled-down version of full environment
- Quick scale-up to full capacity

**4. Hot Standby (Active-Active):**
- Full production capacity in multiple regions
- Zero downtime, automatic failover
- Highest cost and complexity

### Failover Mechanisms
**DNS-based Failover:**
- Update DNS records to point to backup region
- Propagation time (TTL) impacts RTO

**Load Balancer Failover:**
- Health checks trigger traffic redirection
- Cross-region ALB configuration

**Database Failover:**
- Automatic failover with synchronous replication
- Multi-AZ deployments (RDS, Cosmos DB)

## DevOps Integration

### Infrastructure as Code
**Principles:**
- Treat infrastructure like application code
- Version control, code review, automated testing
- Idempotent operations (safe to run multiple times)

**Tools:**
- **Terraform**: Declarative, stateful management
- **CloudFormation**: Native AWS resource management
- **Pulumi**: Imperative with full programming languages

### CI/CD Pipelines
**Pipeline Stages:**
1. **Source**: Code checkout, security scanning
2. **Build**: Compile, unit tests, integration tests
3. **Artifact**: Store immutable artifacts
4. **Deploy**: Progressive rollout (canary, blue-green)
5. **Verify**: Automated tests, monitoring validation

**Advanced Patterns:**
- **GitOps**: Infrastructure changes through Git
- **Deployment Strategies**: Rolling updates, immutable deployments
- **Testing**: Contract testing, chaos engineering

### Monitoring & Observability
**Three Pillars:**
- **Metrics**: Quantitative measurements (Prometheus, CloudWatch)
- **Logs**: Structured debugging information (ELK, Loki)
- **Traces**: Distributed request tracking (Jaeger, Zipkin)

**SRE Principles:**
- **SLIs/SLOs/SLAs**: Service reliability targets
- **Error Budgets**: Explicit failure allowance
- **Blameless Post-mortems**: Learning-focused incident reviews

---

## Key Takeaways

1. **Always consider trade-offs**: No perfect solution exists
2. **Start with requirements**: Scale for actual needs, not hypothetical
3. **Quantify everything**: Back-of-envelope calculations demonstrate engineering thinking
4. **Plan for failure**: Assume things will break, design resilient systems
5. **Security first**: Integrate security in design, not as afterthought
6. **Monitor aggressively**: Measure, instrument, and alert on everything
7. **Iterate and improve**: Systems evolve, design for adaptability

Remember: System design is about **trade-offs** and **justification**. Every decision has costs and benefits—be prepared to explain why you chose a particular approach.
