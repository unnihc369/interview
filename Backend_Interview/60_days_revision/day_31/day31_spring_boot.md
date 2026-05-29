# Day 31 — Resilience4j: Circuit Breaker & Retry

**Topics:** Fault Tolerance · Circuit States · Retry Policies · Spring Integration

---

# 1. Why Resilience4j?

Microservices fail (network, timeouts, overload). Patterns:

- **Circuit Breaker** — stop calling failing service
- **Retry** — transient failure recovery
- **Rate Limiter** — throttle calls
- **Bulkhead** — isolate thread pools
- **Time Limiter** — enforce timeouts

Resilience4j is lightweight, functional, replaces Hystrix (maintenance mode).

---

# 2. Dependencies

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

---

# 3. Circuit Breaker States

```text
CLOSED    → calls pass through; failures counted
    ↓ failure rate > threshold
OPEN      → calls fail fast (CallNotPermittedException)
    ↓ wait duration elapsed
HALF_OPEN → trial calls allowed
    ↓ success → CLOSED | failure → OPEN
```

---

# 4. Configuration

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 10s
        failureRateThreshold: 50
        slowCallRateThreshold: 80
        slowCallDurationThreshold: 2s

  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 500ms
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.net.SocketTimeoutException
          - org.springframework.web.client.HttpServerErrorException
```

---

# 5. Annotation Usage

```java
@Service
public class PaymentClient {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
    @Retry(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<PaymentResponse> charge(PaymentRequest req) {
        return CompletableFuture.supplyAsync(() ->
            restTemplate.postForObject("/charge", req, PaymentResponse.class));
    }

    private CompletableFuture<PaymentResponse> fallback(
            PaymentRequest req, Throwable t) {
        return CompletableFuture.completedFuture(
            PaymentResponse.failed("Payment unavailable"));
    }
}
```

**Order:** Retry → CircuitBreaker → TimeLimiter (configure via aspects).

---

# 6. Programmatic API

```java
CircuitBreaker cb = CircuitBreaker.of("payment",
    CircuitBreakerConfig.custom()
        .failureRateThreshold(50)
        .waitDurationInOpenState(Duration.ofSeconds(10))
        .slidingWindowSize(10)
        .build());

Supplier<PaymentResponse> decorated = CircuitBreaker
    .decorateSupplier(cb, () -> callPaymentApi());

PaymentResponse result = Try.ofSupplier(decorated)
    .recover(throwable -> PaymentResponse.failed())
    .get();
```

---

# 7. Retry Details

```text
maxAttempts: 3        → 1 initial + 2 retries
waitDuration: 500ms  → fixed delay between attempts
retryExceptions       → only these trigger retry
ignoreExceptions      → never retry (e.g., 400 Bad Request)
```

**Idempotency:** Only retry safe/idempotent operations (GET, PUT with idempotency key). POST charge without idempotency key is risky.

---

# 8. Combined with Spring Cloud Gateway

```yaml
spring.cloud.gateway.routes[0].filters:
  - name: CircuitBreaker
    args:
      name: orderService
      fallbackUri: forward:/fallback
```

Gateway uses Resilience4j under the hood in recent Spring Cloud versions.

---

# 9. Monitoring

```properties
management.endpoints.web.exposure.include=health,metrics
management.health.circuitbreakers.enabled=true
```

Metrics: `resilience4j.circuitbreaker.calls`, state transitions, failure rate.

---

# Interview Questions

## Circuit breaker vs retry?

Retry helps **transient** failures. Circuit breaker stops hammering **persistent** failures and allows recovery time.

## Half-open state purpose?

Test if downstream recovered with limited traffic before fully closing circuit.

## Hystrix vs Resilience4j?

Hystrix discontinued. Resilience4j smaller footprint, no thread pool required (can use semaphores), better Java 8+ integration.

## Fallback best practices?

Return cached data, default value, or graceful degradation — never hide errors silently in critical paths without logging.

## Bulkhead use case?

Separate thread pool for payment vs catalog — payment slowness won't exhaust threads for catalog.

---

# Quick Revision

```text
CLOSED → OPEN → HALF_OPEN → CLOSED
@CircuitBreaker + fallbackMethod
@Retry — idempotent ops only
failureRateThreshold + slidingWindowSize
Resilience4j replaces Hystrix in Spring Cloud
```

---

*End of Day 31 Resilience4j*
