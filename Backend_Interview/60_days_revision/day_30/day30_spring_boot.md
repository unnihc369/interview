# Day 30 — Spring Cloud Gateway

**Topics:** API Gateway · Routing · Filters · Rate Limiting · Interview Questions

---

# 1. Why API Gateway?

Microservices expose many endpoints. Clients need:

- Single entry point (SSL termination, routing)
- Cross-cutting concerns: auth, rate limit, logging, CORS
- Protocol translation (HTTP → gRPC)
- Request aggregation (BFF pattern)

**Spring Cloud Gateway** — reactive gateway built on Spring WebFlux + Netty.

---

# 2. Architecture

```text
Client
  ↓
Spring Cloud Gateway (routes, filters, predicates)
  ↓
┌────────────┬────────────┬────────────┐
│ User Svc   │ Order Svc  │ Payment Svc│
└────────────┴────────────┴────────────┘
```

---

# 3. Dependency & Bootstrap

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```properties
server.port=8080
spring.application.name=api-gateway
```

---

# 4. Route Configuration (YAML)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service   # load-balanced via Eureka/Consul
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1          # remove /api before forwarding
            - AddRequestHeader=X-Gateway, true

        - id: user-service
          uri: http://localhost:8081
          predicates:
            - Path=/api/users/**
            - Method=GET,POST
          filters:
            - RewritePath=/api/users/(?<segment>.*), /users/${segment}
```

---

# 5. Predicates (Match Rules)

| Predicate | Example |
|-----------|---------|
| `Path` | `/api/**` |
| `Method` | `GET`, `POST` |
| `Header` | `X-Request-Id, \d+` |
| `Query` | `version, v2` |
| `Host` | `**.example.com` |
| `Cookie` | `session, .+` |
| `After` / `Before` | time-based routing |

---

# 6. Filters

**GatewayFilter** — single route  
**GlobalFilter** — all routes

```java
@Component
public class AuthGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");
        if (token == null || !isValid(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() { return -100; }
}
```

Built-in filters: `Retry`, `CircuitBreaker`, `RequestRateLimiter`, `DedupeResponseHeader`.

---

# 7. Rate Limiting (Redis)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: limited-route
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@ipKeyResolver}"
```

```java
@Bean
public KeyResolver ipKeyResolver() {
    return exchange -> Mono.just(
        exchange.getRequest().getRemoteAddress().getAddress().getHostAddress());
}
```

---

# 8. Gateway vs Zuul

| | Spring Cloud Gateway | Netflix Zuul 1 |
|---|---------------------|----------------|
| Model | Reactive (WebFlux) | Servlet (blocking) |
| Maintenance | Active | Maintenance mode |
| Performance | Better under high concurrency | Thread-per-request limit |

Zuul 2 is async but Gateway is default in Spring Cloud.

---

# 9. Circuit Breaker Integration

```yaml
filters:
  - name: CircuitBreaker
    args:
      name: orderServiceCB
      fallbackUri: forward:/fallback/orders
```

Fallback controller returns cached/default response when downstream fails.

---

# Interview Questions

## Gateway vs Load Balancer?

LB distributes traffic to instances (L4/L7). Gateway adds application-level routing, auth, transformation. Often used together: ALB → Gateway → services.

## How does `lb://service-name` work?

Integrates with Spring Cloud LoadBalancer + service registry (Eureka). Resolves name to instance list, round-robin/weighted selection.

## Reactive — why Netty?

Non-blocking I/O handles many concurrent connections with fewer threads. Good for gateway I/O-bound workload.

## CORS at gateway?

```yaml
spring.cloud.gateway.globalcors.cors-configurations.[/**]:
  allowedOrigins: "*"
  allowedMethods: GET,POST,PUT,DELETE
```

Centralize CORS instead of per-service.

## Order of filters?

`Ordered` interface / `order` attribute. Lower value = higher priority. Auth typically runs early.

---

# Quick Revision

```text
Route = id + uri + predicates + filters
lb://service → discovery + load balancer
StripPrefix, RewritePath → URL mapping
GlobalFilter → auth, logging
RequestRateLimiter → Redis token bucket
Reactive WebFlux + Netty
```

---

*End of Day 30 Spring Cloud Gateway*
