# Phase 7: Message Queues & Async Processing — Lab Exercises

## Lab 1: Kafka on Kubernetes (Strimzi)
```bash
# Install Strimzi operator
kubectl create namespace kafka
kubectl apply -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka

# Create Kafka cluster
cat <<EOF | kubectl apply -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata: { name: my-cluster, namespace: kafka }
spec:
  kafka:
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
    config:
      default.replication.factor: 3
      min.insync.replicas: 2
    storage: { type: jbod, volumes: [{ id: 0, type: persistent-claim, size: 10Gi }] }
  zookeeper:
    replicas: 3
    storage: { type: persistent-claim, size: 5Gi }
EOF

# Create topic
cat <<EOF | kubectl apply -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata: { name: orders, namespace: kafka, labels: { strimzi.io/cluster: my-cluster } }
spec:
  partitions: 12
  replicas: 3
EOF

# Produce and consume
kubectl -n kafka run producer --rm -it --image=quay.io/strimzi/kafka:latest-kafka-3.6.0 -- \
  bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic orders

kubectl -n kafka run consumer --rm -it --image=quay.io/strimzi/kafka:latest-kafka-3.6.0 -- \
  bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 \
  --topic orders --from-beginning --group my-group
```

## Lab 2: Consumer Lag Monitoring
```bash
# Check consumer lag
kubectl -n kafka exec -it my-cluster-kafka-0 -- \
  bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group my-group

# Output:
# TOPIC    PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# orders   0          100             150             50
# orders   1          200             200             0

# Alert if LAG > 1000 → scale consumers or investigate
```

## Lab 3: DLQ Pattern Design
```
Design a DLQ flow for order processing:

  Main Queue: order-events
  DLQ: order-events-dlq

  Flow:
  1. Consumer reads from order-events
  2. Process order (validate, charge, update DB)
  3. If exception → retry 3 times with backoff (1s, 5s, 30s)
  4. After 3 failures → send to order-events-dlq
  5. Alert: DLQ depth > 0 → PagerDuty
  6. Investigate: read DLQ messages, identify root cause
  7. Fix: deploy fix, redrive DLQ messages back to main queue

  Monitoring:
    - DLQ message count (should be 0)
    - DLQ oldest message age
    - Main queue consumer lag
    - Processing error rate
```

## Lab 4: Async Architecture Design Exercise
```
Scenario: E-commerce order pipeline

  Sync path (user-facing, fast):
    POST /order → validate → save to DB (pending) → return 202
                                    ↓
                              Publish "order.created" to Kafka

  Async path (background):
    Payment Service:  consumes order.created → charge card → publish payment.success
    Inventory Service: consumes payment.success → reserve stock → publish inventory.reserved
    Notification Service: consumes inventory.reserved → send confirmation email
    
  Failure handling:
    payment.failed → publish order.cancelled → refund saga
    inventory.failed → publish payment.refund → compensation

  Scaling:
    12 partitions per topic
    Peak: 1000 orders/min → 12 consumers per service
    Normal: 100 orders/min → 3 consumers (auto-scale via KEDA)
```