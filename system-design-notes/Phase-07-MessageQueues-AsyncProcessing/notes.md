# Phase 7: Message Queues & Async Processing — Notes

## 1. Why Async Processing?

```
Sync:  Client → API → Process → DB → Response
  Problem: If processing takes 30s, client waits 30s
  Problem: If downstream is slow/down, entire request fails

Async: Client → API → Queue → Response(202 Accepted)
                         ↓
                      Worker → Process → DB
  Benefits:
    1. Fast response (decouple producer from consumer)
    2. Buffer traffic spikes (queue absorbs burst)
    3. Retry failed processing (message stays in queue)
    4. Scale consumers independently
```

## 2. Message Queue vs Event Stream

```
Message Queue (RabbitMQ, SQS):
  - Point-to-point: message consumed by ONE consumer
  - Message deleted after processing
  - Good for: task distribution, work queues
  - Ordering: per-queue FIFO (SQS FIFO)

Event Stream (Kafka, Kinesis):
  - Pub/Sub: event consumed by MANY consumers
  - Events retained for configured period (7 days default)
  - Good for: event sourcing, CDC, analytics pipelines
  - Ordering: per-partition guaranteed
  - Consumer groups: each group gets all messages

Comparison:
  Feature          Queue (SQS)        Stream (Kafka)
  Consumption      Destructive         Non-destructive
  Replay           No                  Yes (seek to offset)
  Ordering         Queue-level         Partition-level
  Throughput       ~3K msg/s           ~1M msg/s
  Retention        14 days max         Configurable/infinite
  Use case         Task processing     Event pipeline
```

## 3. Apache Kafka Deep Dive

```
Architecture:
  Producer → Topic → Partition → Consumer Group
  
  Topic: logical channel (e.g., "orders")
  Partition: ordered, immutable log (parallelism unit)
  Offset: position in partition (consumer tracks progress)
  Consumer Group: each partition consumed by 1 consumer in group
  Replication Factor: copies across brokers (RF=3 typical)

Key concepts:
  Partition count = max parallelism
    12 partitions → max 12 consumers in a group
  
  Partition key: determines which partition
    OrderID → hash → partition 7
    Same key always → same partition → ordering guaranteed

  ISR (In-Sync Replicas):
    Replicas caught up with leader
    acks=all: write confirmed only when all ISR acknowledge
    min.insync.replicas=2: at least 2 replicas must ACK

Producer delivery semantics:
  at-most-once:   acks=0 (fire and forget, may lose)
  at-least-once:  acks=all + retries (may duplicate)
  exactly-once:   idempotent producer + transactional API
```

## 4. RabbitMQ

```
Exchange types:
  Direct:   Route by exact routing key
  Fanout:   Broadcast to all bound queues
  Topic:    Route by pattern (order.* matches order.created)
  Headers:  Route by message headers

Features:
  - Message acknowledgement (manual/auto)
  - Dead letter exchange (failed messages → DLX)
  - TTL per message or queue
  - Priority queues
  - Delayed messages (plugin)

When to use over Kafka:
  - Complex routing logic needed
  - Message-level priority
  - Smaller scale (<10K msg/s)
  - Task queues (Celery + RabbitMQ)
```

## 5. AWS SQS

```
Standard Queue:
  - At-least-once delivery
  - Best-effort ordering
  - Nearly unlimited throughput
  - Visibility timeout: message hidden while processing

FIFO Queue:
  - Exactly-once processing
  - Strict ordering (per message group)
  - 3,000 msg/s with batching
  - Message deduplication (5 min window)

Dead Letter Queue (DLQ):
  - Messages that fail maxReceiveCount times → DLQ
  - Monitor DLQ depth for alerting
  - Redrive: replay DLQ messages back to source queue

Best practices:
  - Use long polling (WaitTimeSeconds=20) to reduce API calls
  - Batch: SendMessageBatch (up to 10 messages)
  - Set appropriate visibility timeout (> processing time)
```

## 6. Patterns

```
Dead Letter Queue (DLQ):
  Message fails processing X times → moved to DLQ
  DLQ alert → investigate → fix → redrive
  Every queue should have a DLQ

Backpressure:
  Producer faster than consumer → queue grows
  Solutions:
    1. Scale consumers (auto-scale on queue depth)
    2. Rate limit producers
    3. Drop/reject messages (if acceptable)
    4. Circuit breaker on producer side

Idempotency:
  at-least-once = possible duplicates
  Consumer must be idempotent:
    - Use unique message ID
    - Check "already processed" table before processing
    - Upsert instead of insert

Saga Pattern:
  Distributed transaction across services via events
  Order → Payment → Inventory → Shipping
  Each step: do action + publish event
  Failure: publish compensation event (undo)
  Orchestration (central coordinator) vs Choreography (events)
```

---

> **DevOps focus**: Kafka cluster operations, consumer lag monitoring, DLQ alerting, auto-scaling consumers based on queue depth.