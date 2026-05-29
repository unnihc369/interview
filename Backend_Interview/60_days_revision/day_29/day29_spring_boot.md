# Day 29 — Eureka Service Discovery

**Topics:** Microservices Registry · Eureka Server/Client · Load Balancing · Interview Questions

---

# 1. Problem Service Discovery Solves

In microservices, instances scale dynamically (IP/port change). Clients cannot hardcode endpoints.

**Service Discovery** maintains a registry of healthy service instances; clients resolve logical name → actual instances.

---

# 2. Netflix Eureka Architecture

```text
┌─────────────────┐     register/heartbeat     ┌─────────────────┐
│  Order Service  │ ─────────────────────────► │ Eureka Server   │
│  (Eureka Client)│ ◄───────────────────────── │  (Registry)     │
└─────────────────┘     fetch registry         └────────┬────────┘
                                                          │
┌─────────────────┐     lookup "payment-service"        │
│  API Gateway /  │ ◄───────────────────────────────────┘
│  Order Service  │
└─────────────────┘
```

---

# 3. Eureka Server Setup

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```properties
# application.properties (standalone server)
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

---

# 4. Eureka Client (Service Registration)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableDiscoveryClient  // or @EnableEurekaClient
public class OrderServiceApplication { }
```

```properties
spring.application.name=order-service
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
eureka.instance.prefer-ip-address=true
```

On startup, client registers with Eureka and sends **heartbeats** every 30s (default).

---

# 5. Client-Side Load Balancing

With Eureka + Spring Cloud LoadBalancer:

```java
@RestController
public class OrderController {
    private final RestTemplate restTemplate;

    @GetMapping("/orders/{id}/payment")
    public PaymentDto getPayment(@PathVariable Long id) {
        // "payment-service" resolved via Eureka
        return restTemplate.getForObject(
            "http://payment-service/api/payments/for-order/" + id,
            PaymentDto.class);
    }
}

@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

Modern approach: **Spring Cloud OpenFeign**

```java
@FeignClient(name = "payment-service")
public interface PaymentClient {
    @GetMapping("/api/payments/for-order/{orderId}")
    PaymentDto getPayment(@PathVariable Long orderId);
}
```

---

# 6. Registration Lifecycle

```text
1. STARTING   → client boots, not yet registered
2. UP         → registered + heartbeats OK
3. DOWN       → missed heartbeats (self-preservation may delay eviction)
4. CANCELLED  → graceful shutdown (deregister)
```

**Eviction:** Server removes instances with no heartbeat (default ~90s after last heartbeat).

**Self-preservation:** During network partition, Eureka may avoid mass eviction to prevent empty registry.

---

# 7. Eureka vs Alternatives

| Feature | Eureka | Consul | Kubernetes DNS |
|---------|--------|--------|----------------|
| AP model | Yes (availability) | CP option | Infra-native |
| Health checks | Client heartbeat | HTTP/TCP/gRPC | Kube probes |
| Config store | No | Yes (KV) | ConfigMaps |
| Best for | Spring Cloud legacy | Multi-language | K8s deployments |

---

# 8. High Availability Eureka

```text
Peer-replicated Eureka cluster (2+ nodes):

eureka1: register-with-eureka=true, fetch-registry=true
         defaultZone=http://eureka2:8762/eureka/

eureka2: defaultZone=http://eureka1:8761/eureka/
```

Clients register with one peer; registry replicates to peers.

---

# Interview Questions

## Eureka AP vs Consul CP?

Eureka favors **availability** — may serve stale instance list during partition. Consul can enforce consistency; better when stale routing is unacceptable.

## What happens if Eureka is down?

New instances cannot register; existing clients use cached registry until TTL expires. Running instances with cached list may still call known peers.

## `@EnableDiscoveryClient` vs `@EnableEurekaClient`?

`DiscoveryClient` is abstraction (Eureka, Consul, etc.). `@EnableEurekaClient` is Eureka-specific; deprecated in favor of discovery client.

## Heartbeat vs Health Check?

Eureka uses client-driven heartbeats. Does not probe `/health` by default unless integrated with Spring Boot health indicator via custom metadata.

## Is Eureka still recommended?

For new K8s-native projects, **Kubernetes Service + DNS** often replaces Eureka. Eureka remains common in Spring Cloud brownfield and tutorials.

---

# Quick Revision

```text
Eureka Server  → registry (8761)
Eureka Client  → register + heartbeat + fetch
@LoadBalanced  → RestTemplate resolves service name
FeignClient    → declarative HTTP with discovery
Self-preservation → avoids mass eviction during outages
```

---

*End of Day 29 Eureka Service Discovery*
