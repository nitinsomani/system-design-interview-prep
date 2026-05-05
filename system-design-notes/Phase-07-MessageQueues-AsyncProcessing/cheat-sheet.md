# Phase 7: Message Queues & Async Processing — Cheat Sheet

## Queue vs Stream
```
Queue (SQS/RabbitMQ): One consumer, destructive read, task distribution
Stream (Kafka):       Many consumers, replay, event pipeline, high throughput
```

## Kafka Quick Reference
```
Topic:       Logical channel
Partition:   Ordered log, parallelism unit
Offset:      Position in partition
Consumer Group: Each partition → 1 consumer
RF=3:        3 copies across brokers
acks=all:    All ISR must ACK write

Partition count = max consumers in a group
Same partition key → same partition → ordering guaranteed
```

## Delivery Guarantees
```
at-most-once:   May lose, no duplicates (acks=0)
at-least-once:  No loss, may duplicate (acks=all + retries)
exactly-once:   No loss, no duplicates (idempotent + transactions)
```

## SQS
```
Standard:  Unlimited throughput, at-least-once, best-effort order
FIFO:      3K msg/s, exactly-once, strict order per group
DLQ:       Failed messages after maxReceiveCount retries
Long poll: WaitTimeSeconds=20 (reduce API calls)
```

## Key Patterns
```
DLQ:          Failed msgs → DLQ → alert → investigate → redrive
Backpressure: Scale consumers, rate limit producers, circuit breaker
Idempotency:  Dedup by message ID, upsert not insert
Saga:         Distributed txn via events + compensation
```

## Monitoring
```
Kafka:  Consumer lag (offset diff), under-replicated partitions
SQS:    ApproximateNumberOfMessages, ApproximateAgeOfOldestMessage
Alert:  Consumer lag > threshold, DLQ depth > 0
```