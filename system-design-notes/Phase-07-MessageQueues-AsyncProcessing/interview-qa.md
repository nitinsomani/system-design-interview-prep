# Phase 7: Message Queues & Async Processing — Interview Q&A

## Q1: When would you use Kafka vs SQS vs RabbitMQ?
**Answer:** **Kafka**: Event streaming, CDC pipelines, log aggregation, high throughput (>100K msg/s), need replay capability, multiple consumers for same events. **SQS**: Simple task queues, serverless (Lambda triggers), no operational overhead, AWS-native. **RabbitMQ**: Complex routing (topic/header-based), message priority, smaller scale with rich features, Celery task queues. Rule of thumb: Kafka for event pipelines, SQS for simple queues in AWS, RabbitMQ for complex routing at moderate scale.

## Q2: How do you ensure exactly-once processing?
**Answer:** Two levels: 1) **Delivery**: Kafka idempotent producer (enable.idempotence=true) + transactional API ensures exactly-once delivery. SQS FIFO has built-in deduplication (5 min window). 2) **Processing**: Consumer must be idempotent regardless. Strategies: store processed message IDs in DB, use upsert (INSERT ON CONFLICT UPDATE), use DB transaction that includes both processing + offset commit. In practice: most systems use at-least-once delivery + idempotent consumers (simpler than exactly-once).

## Q3: What happens when a Kafka consumer falls behind?
**Answer:** Consumer lag increases (offset behind latest). Monitoring: track lag via Burrow or `kafka-consumer-groups.sh --describe`. Causes: slow processing, insufficient consumers, GC pauses. Solutions: 1) Add consumers (up to partition count). 2) Increase partitions (can't decrease). 3) Optimize processing speed. 4) If lag is extreme: consider skipping old messages or consuming from latest. Risk: if lag > retention period, data loss. DevOps: alert on consumer lag > threshold, auto-scale KEDA on lag metric.

## Q4: Explain the saga pattern for distributed transactions.
**Answer:** Problem: microservices can't use DB transactions across services. Saga breaks the transaction into local transactions + compensation events. Example: Order saga: 1) Create order (pending). 2) Reserve payment → success → reserve inventory → success → confirm order. If inventory fails → compensate: refund payment, cancel order. Two styles: **Choreography**: each service listens for events and acts (simple, decentralized). **Orchestration**: central coordinator directs the flow (easier to understand, single point of control). Prefer orchestration for complex flows (>4 steps).

## Q5: How do you handle message ordering in a distributed system?
**Answer:** Total ordering across all messages is impractical at scale. Instead: **partition-level ordering**. Same partition key → same partition → ordered. Example: all events for user_id=123 go to same partition → processed in order. For SQS FIFO: MessageGroupId provides per-group ordering. Challenge: if you need cross-key ordering (e.g., user A then user B), you need single partition (kills parallelism) or application-level sequencing (timestamps + reconciliation).

## Rapid-Fire
- **Visibility timeout?** → SQS: message hidden from other consumers while being processed
- **Consumer group?** → Kafka: set of consumers sharing partitions; each partition → 1 consumer
- **Fan-out?** → One event consumed by multiple independent consumers (SNS → multiple SQS)
- **Backpressure?** → Producer overwhelms consumer → scale consumers or rate-limit producer
- **DLQ?** → Dead letter queue: messages that fail processing N times → investigate and redrive
- **KEDA?** → K8s event-driven autoscaler: scale pods based on queue depth/consumer lag