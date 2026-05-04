# System Design Interview Questions & Answers

## Design a URL Shortener (like bit.ly)

### Requirements
- Generate short URLs from long URLs
- Redirect short URLs to original URLs
- Handle high read:write ratio (100:1 typical)
- Analytics tracking (clicks, geography)
- Custom URLs (vanity URLs)
- Expiration of URLs

### Scale Estimations
- 100M URLs generated/day
- Peak QPS: 1M reads, 10K writes
- Storage: 100M URLs/day × 365 days × 5 years = 18.25B URLs
- URL storage per URL: 500 bytes (metadata, analytics)

### API Design
```http
POST /api/v1/shorten
{
  "longUrl": "https://example.com/very/long/path",
  "customAlias": "my-link",  # optional
  "expiration": 86400        # seconds
}

GET /:shortCode  # Returns 302 redirect
```

### System Components
1. **Frontend**: Web/React application
2. **API Gateway**: Rate limiting, authentication
3. **URL Generation Service**:
   - Generate unique short codes (base62 encoding)
   - Collision detection and resolution
4. **Storage Layer**:
   - **Primary DB**: PostgreSQL/MySQL (transactional consistency)
   - **Caching**: Redis (URL mappings, rate limits)
   - **Analytics DB**: ClickHouse/TimeScaleDB
5. **CDN**: Global distribution for redirects

### Architecture Patterns
- **Data Partitioning**: Hash-based partitioning by shortCode
- **Write Path**: API → Validation → Generate code → Store in DB + Cache
- **Read Path**: CDN → Cache → Database + Analytics tracking
- **High Availability**: Multi-region deployment, cross-region replication

### Technologies
- **Database**: PostgreSQL with partitioning
- **Cache**: Redis Cluster for high availability
- **Analytics**: Kafka for event streaming, ClickHouse for analytics queries
- **Deployment**: Kubernetes with Istio service mesh

---

## Design Instagram (Photo-sharing platform)

### Requirements
- Upload photos with captions
- News feed for each user
- Follow/unfollow users
- Like/comment on posts
- Timeline: chronological posts from followed users
- Real-time notifications
- Mobile app support

### Scale Estimations
- 1B users, 500M daily active users
- 100M photos uploaded/day
- Peak QPS: 10K uploads, 100K feed requests
- Storage: 100M photos/day × 30KB avg × 5 years = ~15PB data

### System Components
1. **User Service**: Profile management, followers/following
2. **Post Service**: Photo uploads, metadata storage
3. **Feed Service**: Generate user timelines
4. **Notification Service**: Real-time push notifications
5. **Storage System**: Distributed file storage + metadata DB

### Architecture Patterns
- **Fan-out on Write**: When user posts, write to followed users' timelines
- **Fan-out on Read**: Generate timeline on demand for users with many followers
- **Hybrid Approach**: Famous users get fan-out on write, normal users fan-out on read

### Technologies
- **Storage**
  - **Metadata DB**: Cassandra/DynamoDB (NoSQL for horizontal scaling)
  - **Blob Storage**: S3/GCP Cloud Storage
  - **CDN**: CloudFront/Akamai for global photo delivery
  - **Cache**: Redis/Memcached for hot photos

- **Realtime Components**
  - **Message Queue**: Kafka for async processing
  - **WebSocket**: For real-time notifications
  - **Push Service**: FCM/APNS for mobile notifications

- **Processing**
  - **Image Processing**: AWS Lambda for resizing, compression
  - **Recommendation Engine**: ML models for content suggestions

---

## Design WhatsApp (Real-time messaging)

### Requirements
- Send/receive text messages instantly
- Media sharing (photos, videos, documents)
- Group chats with multiple participants
- End-to-end encryption
- Offline message delivery
- Cross-platform synchronization
- Read receipts and typing indicators

### Scale Estimations
- 2B users, 1B daily active users
- 100B messages/day
- Peak QPS: 500K message sends
- Storage: 100B messages/day × 100 bytes = ~3.6PB/day

### System Components
1. **Client Apps**: Android, iOS, Web, Desktop
2. **Message Gateway**: Accept/send messages
3. **Message Processing**: Route, store, encrypt messages
4. **Push Service**: Wake up offline clients
5. **Media Service**: Handle file uploads/downloads

### Architecture Patterns
- **Message Queue**: Buffer messages for offline delivery
- **Dual Write Pattern**: Write to sender's outbox and receiver's inbox
- **Eventually Consistent**: Messages may arrive out of order briefly
- **Geographic Sharding**: Message routing based on user location

### Technologies
- **Communication**: WebSocket + HTTP/2 for real-time transport
- **Database**
  - **Messages**: Cassandra with user-based partitioning
  - **User Data**: MySQL/PostgreSQL for user profiles, groups
- **Caching**: Redis for session data, user presence
- **Encryption**: Signal Protocol for E2E encryption
- **Scaling**: Kubernetes auto-scaling, multi-region deployment

---

## Design Uber (Ride-hailing platform)

### Requirements
- Rider requests ride with pickup/dropoff locations
- Driver accepts/rejects rides
- Real-time tracking of driver and rider location
- Payment processing after ride completion
- Driver/rider matching algorithm
- Surge pricing during high demand
- Safety features and driver background checks

### Scale Estimations
- 100M riders, 5M drivers
- 1M rides/day peaks at 10K concurrent rides
- Location updates: 5M drivers × 30 seconds ≈ 166K QPS
- Storage: Historical rides, location data

### System Components
1. **Location Service**: Track driver/rider positions
2. **Matching Service**: Match riders with nearby drivers
3. **Dispatch Service**: Handle ride lifecycle
4. **Payment Service**: Process payments securely
5. **Analytics Service**: Route optimization, demand forecasting

### Architecture Patterns
- **Geo-hash Indexing**: Efficient location-based queries
- **Event Sourcing**: Ride state as sequence of events
- **CQRS**: Separate read/write models for ride data
- **Saga Pattern**: Distributed transactions for ride booking

### Technologies
- **Location Data**
  - **Geo-indexing**: Redis Geospatial, Elasticsearch with Geo plugins
  - **Real-time Processing**: Kafka Streams for location updates

- **Matching Algorithm**
  - **Search**: Quadtree/GeoHash for nearby drivers
  - **Optimization**: ML models for ETAs, routing

- **Payment**
  - **PCI Compliant**: Secure payment processing
  - **Settlement**: Batch processing for driver payouts

- **Real-time Features**
  - **WebSocket**: Real-time ride tracking
  - **Push Notifications**: Ride updates, driver arrivals

---

## Design Netflix (Video streaming service)

### Requirements
- Stream movies/TV shows in multiple formats
- HD/4K quality with adaptive bitrate
- Multiple device synchronization
- Personalized recommendations
- Offline downloads
- Global CDN distribution
- Regional content restrictions

### Scale Estimations
- 270M subscribers, 100M daily active users
- Peak concurrent streams: 10M (200M hours/month)
- Video library: 20,000+ titles, 7TB+ storage
- Bandwidth: 100Gbps+ peak global traffic

### System Components
1. **Video Encoding Service**: Multiple codec/quality generation
2. **Content Delivery Network**: Global video distribution
3. **Recommendation Engine**: ML-based personalized suggestions
4. **Playback Service**: Adaptive bitrate streaming
5. **User Service**: Profiles, subscriptions, watch history

### Architecture Patterns
- **Edge Computing**: Video processing close to users
- **Microservices**: Separate services for encoding, streaming, billing
- **Event-driven**: Async processing for video uploads
- **Multi-CDN**: Multiple CDNs for cost optimization

### Technologies
- **Video Processing**
  - **Encoding**: FFmpeg, AWS Elemental MediaConvert
  - **Storage**: S3 for source, CloudFront for delivery

- **Streaming**
  - **Protocol**: HLS/DASH for adaptive streaming
  - **Encryption**: DRM protection (Widevine, FairPlay)

- **Recommendation**
  - **ML Framework**: TensorFlow, PyTorch
  - **Data Pipeline**: Apache Spark for batch processing

- **Infrastructure**
  - **Orchestration**: Kubernetes + Istio
  - **Monitoring**: Prometheus + Grafana

---

## Design a Notification System (like Slack alerts)

### Requirements
- Multiple channels: email, SMS, push notifications, Slack/Teams
- Scheduled vs. immediate notifications
- Retry logic with exponential backoff
- Bounce/dead letter handling
- A/B testing for notification strategies
- Analytics on delivery rates, open rates

### Scale Estimations
- 1B notifications/day (100K QPS)
- Multiple channels with different SLAs
- 98%+ delivery success rate required

### System Components
1. **Notification API**: Receive notification requests
2. **Channel Services**: Email, SMS, Push services
3. **Queue System**: Buffer and prioritize notifications
4. **Analytics Service**: Track delivery metrics

### Architecture Patterns
- **Priority Queues**: High-priority notifications processed first
- **Circuit Breaker**: Fail fast on channel degradation
- **Bulk Processing**: Send multiple emails/SMS per API call
- **Template Service**: Dynamic content generation

### Technologies
- **Queue**: Kafka for high throughput, RabbitMQ for low-latency
- **External Services**: SendGrid (email), Twilio (SMS), FCM/APNS (push)
- **Templates**: Handlebars, Jinja2 for dynamic content
- **Circuit Breaking**: Resilience4j, Hystrix

---

## DevOps-Focused Questions

### How do you handle database migrations in production?

**Approach:**
- Forward-only migrations in production
- Branching strategy: feature branch → develop → staging → master
- Tools: Flyway (SQL), Liquibase (XML), Alembic (Python)
- Rollback strategy: Previous version rollback, data migration backfill

**Best Practices:**
- Test migrations on production-like data
- Break large migrations into smaller chunks
- Use feature flags for gradual rollout
- Monitor migration performance and locks

### Explain blue-green vs. canary deployments

**Blue-Green Deployment:**
- Two identical environments: Blue (current) and Green (next)
- Traffic shifts from Blue to Green instantly
- Pros: Quick rollback, zero downtime
- Cons: Double infrastructure cost, resource waste

**Canary Deployment:**
- Deploy new version to small subset of users/servers
- Gradually increase traffic to new version
- Pros: Risk mitigation, real-user testing
- Cons: Complex traffic routing, monitoring required

### How do you ensure reliability in microservices?

**Circuit Breaker Pattern:**
- Prevent cascading failures
- States: Closed (normal), Open (fail fast), Half-open (test recovery)

**Timeouts and Retries:**
- Exponential backoff for retries
- Jitter to avoid thundering herd
- Idempotent operations for safe retries

**Distributed Tracing:**
- Jaeger/OpenTelemetry for request tracking
- Correlation IDs across service boundaries
- Performance bottleneck identification

### How do you scale a monolithic application?

**Step 1**: Vertical scaling (increase resources) - Quick but limited

**Step 2**: Database optimization
- Indexing, query tuning, connection pooling
- Read replicas, caching layers

**Step 3**: Application layer optimization
- Code profiling, memory leaks
- Asynchronous processing, background jobs

**Step 4**: Horizontal scaling
- Load balancers, session storage
- Stateless application design
- Microservice decomposition

**Step 5**: Data layer scaling
- Database sharding, CQRS pattern
- Event sourcing for audit trails

### Explain database failover strategies

**Automatic Failover:**
- Detective mechanisms: Heartbeat checks, quorum
- Promotion: Standby database becomes primary
- DNS updates: Application routing to new primary

**Manual Failover (Planned Maintenance):**
- Graceful shutdown of primary
- Promote standby with minimal disruption
- Data consistency verification

**Multi-Region Failover:**
- Cross-region read replicas
- DNS-based routing (Route53, Cloudflare)
- Data synchronization strategies

---

## Behavioral Questions

**Tell me about a time you designed a system that failed:**
- Use STAR method (Situation, Task, Action, Result)
- Show learning from failure
- Focus on technical challenges solved

**How do you stay updated with technology trends?**
- Blogs, conferences, open-source contributions
- Continuous learning (courses, certifications)
- Internal tech talks, cross-team collaboration

**Describe a time you had to make a trade-off between speed and correctness:**
- Explain technical constraints
- Business impact of decisions
- Mitigation strategies implemented

---

## Final Tips
- **Quantify everything**: Use back-of-envelope calculations
- **Start simple**: Monolithic first, then optimize bottlenecks
- **Trade-offs**: Always discuss alternatives and why you chose solution
- **Know your tech deep**: Be prepared for follow-up questions on chosen technologies
- **Practice diagramming**: Draw architecture diagrams on whiteboard
