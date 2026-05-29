# Day 19 — SLF4J, Logback & @Slf4j

**Topics:** Logging Facade · Logback Configuration · Log Levels · Structured Logging · Interview Questions

---

# 1. Java Logging Architecture

```text
Application Code
      ↓
SLF4J (facade/API)
      ↓
Logback (implementation)
      ↓
Console / File / External (ELK, CloudWatch)
```

---

# Why SLF4J + Logback?

| Layer | Role |
|-------|------|
| SLF4J | Logging API — write code against interface |
| Logback | Default implementation in Spring Boot |
| Log4j2 | Alternative implementation |

**Benefit:** Swap implementation without changing application code.

---

# 2. Spring Boot Default Setup

Spring Boot includes automatically:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

Includes: SLF4J + Logback + jul-to-slf4j bridge.

---

# 3. Using Logger (Manual)

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class OrderService {

    private static final Logger log = LoggerFactory.getLogger(OrderService.class);

    public Order createOrder(OrderRequest req) {
        log.info("Creating order for user: {}", req.getUserId());
        try {
            Order order = processOrder(req);
            log.debug("Order created: {}", order.getId());
            return order;
        } catch (Exception e) {
            log.error("Failed to create order for user: {}", req.getUserId(), e);
            throw e;
        }
    }
}
```

---

# 4. @Slf4j (Lombok)

Auto-generates `private static final Logger log`.

```java
@Slf4j
@Service
public class OrderService {

    public Order createOrder(OrderRequest req) {
        log.info("Creating order for user: {}", req.getUserId());
        // ...
    }
}
```

Requires Lombok dependency + IDE annotation processing.

---

# 5. Log Levels (Priority Order)

```text
TRACE < DEBUG < INFO < WARN < ERROR
```

| Level | When to Use |
|-------|-------------|
| TRACE | Very detailed debugging |
| DEBUG | Development diagnostics |
| INFO | Normal business events |
| WARN | Unexpected but recoverable |
| ERROR | Failures requiring attention |

---

# 6. Parameterized Logging

**Always use `{}` placeholders — not string concatenation.**

```java
// GOOD — lazy evaluation, no string built if level disabled
log.debug("Processing order {} for user {}", orderId, userId);

// BAD — string concatenation always executes
log.debug("Processing order " + orderId + " for user " + userId);
```

---

# 7. Logback Configuration — application.properties

```properties
# Root log level
logging.level.root=INFO

# Package-specific levels
logging.level.com.example.orders=DEBUG
logging.level.org.springframework.web=WARN
logging.level.org.hibernate.SQL=DEBUG

# Log file
logging.file.name=logs/application.log
logging.file.max-size=10MB
logging.file.max-history=30

# Pattern
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
```

---

# 8. Logback Configuration — logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <springProperty name="APP_NAME" source="spring.application.name"/>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/${APP_NAME}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/${APP_NAME}.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d %level [%thread] %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>

    <logger name="com.example" level="DEBUG"/>
</configuration>
```

Use `logback-spring.xml` (not `logback.xml`) for Spring profile support.

---

# 9. Profile-Specific Logging

```xml
<springProfile name="dev">
    <root level="DEBUG">
        <appender-ref ref="CONSOLE"/>
    </root>
</springProfile>

<springProfile name="prod">
    <root level="WARN">
        <appender-ref ref="FILE"/>
    </root>
</springProfile>
```

---

# 10. Structured Logging (JSON)

For ELK/Splunk/CloudWatch:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

```xml
<appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>
```

Output:

```json
{"timestamp":"2024-05-29T10:00:00","level":"INFO","message":"Order created","orderId":"12345"}
```

---

# 11. MDC — Mapped Diagnostic Context

Attach context to all log lines in a request:

```java
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        MDC.put("requestId", UUID.randomUUID().toString());
        MDC.put("userId", extractUserId(request));
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

Log pattern includes MDC:

```properties
logging.pattern.console=%d [%X{requestId}] [%X{userId}] %-5level %msg%n
```

---

# 12. Change Log Level at Runtime (Actuator)

```bash
curl -X POST http://localhost:8080/actuator/loggers/com.example \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'
```

---

# Interview Questions

## SLF4J vs Log4j?

SLF4J is a **facade** (API only). Log4j/Logback are **implementations**. Code uses SLF4J; binding determines actual logger.

---

## Why not System.out.println?

- No log levels
- No timestamps/thread info
- Cannot redirect to file/external system
- No performance optimization (lazy evaluation)
- Not production-ready

---

## Log level in production?

Typically `INFO` for application code, `WARN` for root. Enable `DEBUG` temporarily for troubleshooting via Actuator.

---

## How to log exceptions properly?

```java
// Include exception as last parameter — prints stack trace
log.error("Payment failed for order {}", orderId, exception);
```

Never: `log.error("Error: " + exception.getMessage())` — loses stack trace.

---

## Performance best practice?

1. Use parameterized logging `{}`
2. Guard expensive debug: `if (log.isDebugEnabled()) { log.debug("Data: {}", expensiveToString()); }`
3. Async appenders for high-throughput apps

---

# Quick Revision

```text
SLF4J     → logging API (facade)
Logback   → default Spring Boot implementation
@Slf4j     → Lombok auto-generates logger
Levels    → TRACE < DEBUG < INFO < WARN < ERROR
{}        → parameterized logging (always use)
MDC       → request-scoped context in all logs
logback-spring.xml → profile-aware config
```

---

*End of Day 19 SLF4J & Logback*
