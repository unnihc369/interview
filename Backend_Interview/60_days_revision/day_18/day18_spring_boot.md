# Day 18 — Spring Boot Actuator

**Topics:** Production Monitoring · Health Checks · Metrics · Custom Endpoints · Interview Questions

---

# 1. What is Spring Boot Actuator?

Production-ready features for monitoring and managing applications.

Provides:
- Health checks
- Metrics
- Environment info
- Application status
- Custom management endpoints

---

# 2. Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

# 3. Default Endpoints

After adding dependency, endpoints available at `/actuator/*`:

| Endpoint | Description |
|----------|-------------|
| `/actuator/health` | Application health status |
| `/actuator/info` | Custom app info |
| `/actuator/metrics` | Available metrics |
| `/actuator/metrics/{name}` | Specific metric |
| `/actuator/env` | Environment properties |
| `/actuator/loggers` | View/modify log levels |
| `/actuator/beans` | All Spring beans |

---

# 4. Configuration

```properties
# application.properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.endpoint.health.show-details=when-authorized
management.info.env.enabled=true
```

---

# Expose Endpoints

By default, only `health` and `info` exposed over HTTP.

```properties
management.endpoints.web.exposure.include=*
# Production: expose only needed endpoints
management.endpoints.web.exposure.include=health,metrics,prometheus
```

---

# 5. Health Indicator

```json
GET /actuator/health

{
  "status": "UP",
  "components": {
    "db": { "status": "UP" },
    "diskSpace": { "status": "UP" },
    "redis": { "status": "DOWN" }
  }
}
```

Overall status = UP only if all components UP.

---

# 6. Custom Health Indicator

```java
@Component
public class PaymentGatewayHealthIndicator implements HealthIndicator {

    private final PaymentClient paymentClient;

    @Override
    public Health health() {
        try {
            paymentClient.ping();
            return Health.up()
                    .withDetail("gateway", "Payment API reachable")
                    .build();
        } catch (Exception e) {
            return Health.down()
                    .withDetail("error", e.getMessage())
                    .build();
        }
    }
}
```

---

# 7. Custom Info Contributor

```java
@Component
public class AppInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("app", Map.of(
                "name", "OrderService",
                "version", "2.1.0",
                "environment", "production"
        ));
    }
}
```

Or via properties:

```properties
info.app.name=OrderService
info.app.version=2.1.0
info.app.description=Handles order processing
```

---

# 8. Metrics with Micrometer

Actuator uses **Micrometer** for metrics abstraction.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final MeterRegistry meterRegistry;

    public Order createOrder(OrderRequest req) {
        Timer.Sample sample = Timer.start(meterRegistry);
        try {
            Order order = processOrder(req);
            meterRegistry.counter("orders.created", "status", "success").increment();
            return order;
        } catch (Exception e) {
            meterRegistry.counter("orders.created", "status", "failure").increment();
            throw e;
        } finally {
            sample.stop(meterRegistry.timer("orders.processing.time"));
        }
    }
}
```

---

# 9. Prometheus Integration

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=prometheus
management.metrics.export.prometheus.enabled=true
```

Scrape endpoint: `/actuator/prometheus`

---

# 10. Securing Actuator Endpoints

```java
@Bean
public SecurityFilterChain actuatorSecurity(HttpSecurity http) throws Exception {
    http.securityMatcher("/actuator/**")
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/actuator/health").permitAll()
            .requestMatchers("/actuator/**").hasRole("ADMIN")
        );
    return http.build();
}
```

Production best practice:
- Expose `health` publicly (for load balancer)
- Protect sensitive endpoints (`env`, `beans`, `loggers`)

---

# 11. Kubernetes Liveness & Readiness

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
```

Enable probes:

```properties
management.endpoint.health.probes.enabled=true
```

---

# Interview Questions

## Why use Actuator over custom health endpoint?

Standardized endpoints, integrates with Micrometer/Prometheus/Grafana, custom health indicators plug in easily, Kubernetes-ready probes.

---

## Difference between liveness and readiness?

| Probe | Purpose |
|-------|---------|
| Liveness | Is app running? Restart if fails |
| Readiness | Can app accept traffic? Remove from load balancer if fails |

---

## How to change log level at runtime?

```bash
curl -X POST http://localhost:8080/actuator/loggers/com.example \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'
```

---

## Actuator vs APM tools (Datadog, New Relic)?

Actuator provides basic metrics/health. APM adds distributed tracing, alerting, dashboards. Often used together — Actuator exports to Prometheus, APM adds tracing.

---

## What metrics to track in production?

- Request latency (p50, p95, p99)
- Error rate
- JVM memory/GC
- Database connection pool usage
- Custom business metrics (orders/min)

---

# Quick Revision

```text
Actuator → production monitoring endpoints
/health  → UP/DOWN status with components
/metrics → Micrometer metrics
Custom HealthIndicator → check external dependencies
Prometheus → scrape /actuator/prometheus
Secure endpoints → health public, rest admin-only
```

---

*End of Day 18 Spring Boot Actuator*
