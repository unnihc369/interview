# Day 32 — Distributed Tracing: Sleuth & Zipkin

**Topics:** Trace ID · Span · Micrometer Tracing · Interview Questions

---

# 1. Why Distributed Tracing?

Microservice call chain: `API → Order → Payment → Inventory`.

When latency spikes, which service caused it? **Tracing** correlates logs and spans across services with shared **trace ID**.

---

# 2. Core Concepts

| Term | Meaning |
|------|---------|
| Trace | End-to-end request journey |
| Span | Single operation (HTTP call, DB query) |
| Trace ID | Unique ID for entire request |
| Span ID | Unique ID for one operation |
| Parent Span | Caller span linking hierarchy |

```text
Trace abc123
  Span 1: API Gateway (10ms)
    Span 2: Order Service (80ms)
      Span 3: Payment Service (60ms)
      Span 4: DB query (5ms)
```

---

# 3. Spring Cloud Sleuth → Micrometer Tracing

**Sleuth** (legacy) auto-instrumented Spring apps. Modern stack:

- **Micrometer Tracing** (facade)
- **Brave** or **OpenTelemetry** (tracer implementation)
- **Zipkin** or **Jaeger** (collector UI)

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

```properties
management.tracing.sampling.probability=1.0
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
```

---

# 4. Auto-Instrumentation

With tracing on classpath, Spring Boot auto-configures:

- HTTP server/client (RestTemplate, WebClient, Feign)
- Messaging (Kafka, RabbitMQ)
- JDBC (optional)

Headers propagated:

```text
X-B3-TraceId
X-B3-SpanId
X-B3-ParentSpanId
X-B3-Sampled
```

(W3C `traceparent` header in OpenTelemetry mode.)

---

# 5. Manual Spans

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final Tracer tracer;

    public Order createOrder(OrderRequest req) {
        Span span = tracer.nextSpan().name("create-order").start();
        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            span.tag("userId", req.getUserId());
            return orderRepository.save(buildOrder(req));
        } catch (Exception e) {
            span.error(e);
            throw e;
        } finally {
            span.end();
        }
    }
}
```

---

# 6. Zipkin Setup

```bash
docker run -d -p 9411:9411 openzipkin/zipkin
```

UI: `http://localhost:9411` — search traces by service, latency, tags.

---

# 7. Log Correlation

```properties
logging.pattern.level=%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]
```

Logs include trace/span IDs — grep logs by trace ID across services.

---

# 8. Sampling

```properties
# 100% dev, 10% prod
management.tracing.sampling.probability=0.1
```

Always sample errors (custom `Sampler` in Brave).

---

# 9. Sleuth vs OpenTelemetry

| | Brave + Zipkin | OpenTelemetry |
|---|----------------|---------------|
| Ecosystem | Spring-native | Vendor-neutral standard |
| Export | Zipkin format | OTLP to many backends |
| Future | Supported via Micrometer | Industry direction |

---

# Interview Questions

## Trace vs span?

Trace = full tree. Span = one node (service call or internal operation).

## How is trace ID propagated?

HTTP headers (B3 or W3C). Messaging systems copy headers to message metadata. Feign/RestTemplate interceptors add headers automatically.

## Zipkin vs ELK?

ELK aggregates logs. Zipkin visualizes latency breakdown per span. Use together: logs correlated by traceId.

## Performance impact?

Sampling reduces overhead. Async span reporting to collector minimizes blocking.

## Without tracing, how to debug?

Correlated request ID in MDC manually — tracing automates this + timing per span.

---

# Quick Revision

```text
Trace ID → entire request
Span → single operation with parent
Micrometer Tracing + Brave + Zipkin
management.tracing.sampling.probability
Logs: %X{traceId}, %X{spanId}
B3 / traceparent headers propagated
```

---

*End of Day 32 Sleuth & Zipkin Tracing*
