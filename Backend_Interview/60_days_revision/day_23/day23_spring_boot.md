# Day 23 - @Cacheable & Redis Caching

---

# 1. Why Caching?

Backend systems cache to:

- Reduce database load
- Lower latency for hot reads
- Improve throughput under traffic spikes

```text
Client → App → Cache (Redis) → DB (on miss)
```

---

# 2. Spring Cache Abstraction

Spring provides annotation-based caching independent of provider.

Enable caching:

```java
@SpringBootApplication
@EnableCaching
public class Application { }
```

---

# 3. @Cacheable

Caches method **return value** by key. Subsequent calls with same key skip method execution.

```java
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return productRepository.findById(id)
                .orElseThrow(() -> new NotFoundException("Product not found"));
    }
}
```

---

# Key Expressions (SpEL)

| Expression | Meaning |
|------------|---------|
| `#id` | Method parameter |
| `#product.id` | Property of parameter |
| `#root.methodName` | Method name |
| `'all'"` | Constant key |

```java
@Cacheable(value = "products", key = "#root.methodName + '_' + #category")
public List<Product> getByCategory(String category) { ... }
```

---

# 4. Other Cache Annotations

| Annotation | Purpose |
|------------|---------|
| `@CachePut` | Always execute method, update cache |
| `@CacheEvict` | Remove cache entry(s) |
| `@Caching` | Combine multiple cache ops |
| `@CacheConfig` | Class-level default cache names |

```java
@CachePut(value = "products", key = "#product.id")
public Product updateProduct(Product product) {
    return productRepository.save(product);
}

@CacheEvict(value = "products", key = "#id")
public void deleteProduct(Long id) {
    productRepository.deleteById(id);
}

@CacheEvict(value = "products", allEntries = true)
public void refreshAll() { ... }
```

---

# 5. Redis Setup (Spring Boot)

## Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

## application.properties

```properties
spring.data.redis.host=localhost
spring.data.redis.port=6379
spring.cache.type=redis
spring.cache.redis.time-to-live=600000
```

---

# 6. Redis Cache Manager Configuration

```java
@Configuration
@EnableCaching
public class RedisCacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration
                .defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .disableCachingNullValues()
                .serializeValuesWith(
                    RedisSerializationContext.SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer())
                );

        return RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .withCacheConfiguration("products",
                    config.entryTtl(Duration.ofMinutes(30)))
                .build();
    }
}
```

---

# 7. Cache-Aside Pattern (Most Common)

```text
1. Read from cache
2. Cache miss → read DB
3. Write to cache
4. Return data
```

`@Cacheable` implements this automatically on method boundary.

---

# 8. Cache Invalidation Strategies

| Strategy | When |
|----------|------|
| TTL (time-to-live) | Stale data acceptable for N minutes |
| Write-through | Update cache on every write |
| Write-behind | Async cache update |
| Evict on update | `@CacheEvict` on PUT/DELETE |

**Interview quote:** *"There are only two hard things: cache invalidation and naming things."*

---

# 9. Advanced Topics

## Conditional Caching

```java
@Cacheable(value = "products", key = "#id", unless = "#result == null")
```

## Sync Cache (prevent stampede)

```java
@Cacheable(value = "products", key = "#id", sync = true)
```

Only one thread loads value; others wait — avoids thundering herd on hot keys.

## Custom KeyGenerator

```java
@Bean
public KeyGenerator customKeyGenerator() {
    return (target, method, params) ->
        method.getName() + Arrays.deepHashCode(params);
}
```

---

# 10. Redis Data Structures (Interview)

| Structure | Use Case |
|-----------|----------|
| String | Simple key-value cache |
| Hash | Object fields (user profile) |
| List | Recent items, queues |
| Set | Unique tags, online users |
| Sorted Set | Leaderboards, rate limiting |
| Pub/Sub | Real-time notifications (not durable cache) |

---

# 11. Pitfalls

| Pitfall | Solution |
|---------|----------|
| Caching mutable objects | Return immutable copies or DTOs |
| Self-invocation bypasses cache | Call via proxy or separate bean |
| Stale data after update | `@CacheEvict` on write methods |
| Serializing JPA entities | Use DTOs — lazy loading breaks in Redis |
| No TTL | Memory growth in Redis |

---

# Common Interview Questions

## Q1. How does @Cacheable work internally?

AOP proxy intercepts call → checks cache manager → returns cached value or invokes method and stores result.

## Q2. Why self-invocation doesn't cache?

`this.getProduct()` bypasses Spring proxy. Inject self or move to another `@Service`.

## Q3. Redis vs local (Caffeine) cache?

Local: lowest latency, per-instance. Redis: shared across instances, survives restarts (with persistence).

## Q4. Cache penetration vs breakdown vs avalanche?

- **Penetration:** queries for non-existent keys → bloom filter
- **Breakdown:** hot key expires → sync=true or never expire hot keys
- **Avalanche:** mass expiry → random TTL jitter

## Q5. How to cache List responses?

Use composite key: `@Cacheable(key = "#category + '_' + #page")` with pagination params.

## Q6. Is Redis single-threaded?

Command execution is single-threaded per instance (fast), but I/O is multiplexed; use cluster for scale.

---

# One-Line Revision

```text
@EnableCaching + @Cacheable on service methods + RedisCacheManager = distributed read-through cache with TTL and eviction.
```

---

*End of Day 23 Spring Boot*
