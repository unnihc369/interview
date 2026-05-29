# Day 36 — Spring Kafka Producer & Consumer

**Topics:** Kafka Basics · Spring Kafka Config · Producer · Consumer · Error Handling · Interview Questions

---

# 1. Why Kafka in Spring?

| Feature | Benefit |
|---------|---------|
| Durable log | Replay events after failure |
| High throughput | Millions msgs/sec |
| Decoupling | Producers/consumers scale independently |
| Ordering | Per-partition ordering |

Spring Kafka wraps `kafka-clients` with templates, listeners, and Boot auto-config.

---

# 2. Dependencies

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=order-service
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
```

---

# 3. Producer — KafkaTemplate

```java
@Service
@RequiredArgsConstructor
public class OrderEventPublisher {

    private final KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate;

    public void publish(OrderCreatedEvent event) {
        kafkaTemplate.send("order-created", event.orderId(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Failed to publish order {}", event.orderId(), ex);
                } else {
                    log.info("Published to partition {} offset {}",
                        result.getRecordMetadata().partition(),
                        result.getRecordMetadata().offset());
                }
            });
    }
}
```

---

# 4. Producer Config (Advanced)

```java
@Configuration
public class KafkaProducerConfig {

    @Bean
    public ProducerFactory<String, OrderCreatedEvent> producerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.ACKS_CONFIG, "all");           // durability
        props.put(ProducerConfig.RETRIES_CONFIG, 3);
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true); // exactly-once-ish
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return new DefaultKafkaProducerFactory<>(props);
    }

    @Bean
    public KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

---

# 5. Consumer — @KafkaListener

```java
@Service
@Slf4j
public class OrderEventConsumer {

    @KafkaListener(topics = "order-created", groupId = "inventory-service")
    public void handle(OrderCreatedEvent event) {
        log.info("Processing order {}", event.orderId());
        inventoryService.reserve(event);
    }
}
```

---

# 6. Manual Ack & Concurrency

```java
@KafkaListener(
    topics = "order-created",
    groupId = "notification-service",
    containerFactory = "manualAckFactory"
)
public void handle(OrderCreatedEvent event, Acknowledgment ack) {
    try {
        emailService.sendConfirmation(event);
        ack.acknowledge();
    } catch (Exception e) {
        // do not ack — message redelivered
        throw e;
    }
}
```

```java
@Bean
public ConcurrentKafkaListenerContainerFactory<String, OrderCreatedEvent> manualAckFactory(
        ConsumerFactory<String, OrderCreatedEvent> consumerFactory) {
    ConcurrentKafkaListenerContainerFactory<String, OrderCreatedEvent> factory =
        new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory);
    factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);
    factory.setConcurrency(3); // one thread per partition (up to partition count)
    return factory;
}
```

---

# 7. Error Handling — Retry & DLT

```java
@Bean
public DefaultErrorHandler errorHandler(KafkaTemplate<String, Object> template) {
  DeadLetterPublishingRecoverer recoverer =
      new DeadLetterPublishingRecoverer(template,
          (record, ex) -> new TopicPartition(record.topic() + ".DLT", record.partition()));

  DefaultErrorHandler handler = new DefaultErrorHandler(recoverer,
      new FixedBackOff(1000L, 3)); // 3 retries, 1s apart
  handler.addNotRetryableExceptions(IllegalArgumentException.class);
  return handler;
}
```

---

# 8. Idempotent Consumer Pattern

```text
1. Consumer receives event with eventId
2. Check idempotency store (Redis/DB)
3. If processed → ack and skip
4. Else process in transaction + store eventId + ack
```

Prevents duplicate side effects on at-least-once delivery.

---

# 9. Key Interview Concepts

| Term | Meaning |
|------|---------|
| Topic | Named log stream |
| Partition | Ordered shard; parallelism unit |
| Offset | Position in partition |
| Consumer Group | Load-balance partitions across consumers |
| Rebalance | Partition reassignment on join/leave |

---

# Interview Questions

## Q1. At-least-once vs exactly-once?

At-least-once: may duplicate on retry. Exactly-once: transactional producer + idempotent consumer + Kafka transactions (complex).

## Q2. Why `acks=all`?

Leader waits for all in-sync replicas — strongest durability; slower than `acks=1`.

## Q3. Consumer lag?

Difference between latest offset and consumer offset. High lag → slow consumer or under-provisioned.

## Q4. Partition key choice?

Same key → same partition → ordering per entity (e.g. `orderId`).

## Q5. Spring `@KafkaListener` vs raw consumer?

Listener handles threading, ack, deserialization, error handlers — production-ready defaults.

---

# One-Line Revision

```text
KafkaTemplate.send for producers; @KafkaListener for consumers; acks=all + idempotence for durability.
Manual ack + DLT for failures; partition key for per-entity ordering.
```

---

*End of Day 36 Spring Boot*
