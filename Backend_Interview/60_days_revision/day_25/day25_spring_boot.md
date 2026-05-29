# Day 25 - @Conditional & Spring Profiles

---

# 1. Spring Profiles

Profiles activate beans/config for specific environments.

```properties
# application-dev.properties
spring.datasource.url=jdbc:mysql://localhost:3306/devdb

# application-prod.properties
spring.datasource.url=jdbc:mysql://prod-host:3306/proddb
```

Activate profile:

```properties
spring.profiles.active=dev
```

Or CLI: `--spring.profiles.active=prod`

---

# 2. @Profile on Beans

```java
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder().build();
    }
}

@Configuration
@Profile("prod")
public class ProdConfig {
    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl(env.getProperty("spring.datasource.url"));
        return ds;
    }
}
```

```java
@Service
@Profile("!test")
public class EmailService { ... }  // active when NOT test profile
```

---

# 3. @Conditional — Conditional Bean Registration

`@Conditional` registers a bean **only if condition matches**.

Spring Boot provides `@ConditionalOn*` meta-annotations:

| Annotation | Condition |
|------------|-----------|
| `@ConditionalOnProperty` | Property value matches |
| `@ConditionalOnClass` | Class on classpath |
| `@ConditionalOnMissingBean` | No bean of type exists |
| `@ConditionalOnWebApplication` | Running as web app |
| `@ConditionalOnExpression` | SpEL evaluates true |

---

# 4. @ConditionalOnProperty

```java
@Configuration
@ConditionalOnProperty(
    name = "feature.notifications.enabled",
    havingValue = "true",
    matchIfMissing = false
)
public class NotificationConfig {

    @Bean
    public NotificationService notificationService() {
        return new EmailNotificationService();
    }
}
```

```properties
feature.notifications.enabled=true
```

---

# 5. Custom @Conditional

```java
public class OnLinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext ctx, AnnotatedTypeMetadata meta) {
        return System.getProperty("os.name").toLowerCase().contains("linux");
    }
}

@Configuration
@Conditional(OnLinuxCondition.class)
public class LinuxOnlyConfig { }
```

Or annotation wrapper:

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Conditional(OnLinuxCondition.class)
public @interface ConditionalOnLinux { }
```

---

# 6. Profiles vs @Conditional

| | @Profile | @Conditional |
|---|----------|--------------|
| Purpose | Environment-based (dev/prod) | Any custom logic |
| Activation | `spring.profiles.active` | Condition class evaluates |
| Typical use | DB config, mock services | Feature flags, classpath checks |

Often combined:

```java
@Bean
@Profile("prod")
@ConditionalOnProperty("cache.redis.enabled")
public RedisCacheManager cacheManager() { ... }
```

---

# 7. @ConditionalOnMissingBean

Auto-configuration pattern — only create default if user didn't define own:

```java
@Bean
@ConditionalOnMissingBean
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
```

This is how Spring Boot auto-config works internally.

---

# 8. application.yml Multi-Document Profiles

```yaml
spring:
  config:
    activate:
      on-profile: dev
---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:mysql://prod/db
```

---

# 9. Feature Toggle Pattern

```java
@Service
@RequiredArgsConstructor
public class PaymentService {
    private final FeatureFlags flags;

    public void pay(Order order) {
        if (flags.isEnabled("new-payment-gateway")) {
            newGateway.charge(order);
        } else {
            legacyGateway.charge(order);
        }
    }
}
```

Or bean-level:

```java
@Bean
@ConditionalOnProperty(name = "payment.provider", havingValue = "stripe")
public PaymentGateway stripeGateway() { return new StripeGateway(); }

@Bean
@ConditionalOnProperty(name = "payment.provider", havingValue = "razorpay")
public PaymentGateway razorpayGateway() { return new RazorpayGateway(); }
```

---

# 10. Testing with Profiles

```java
@SpringBootTest
@ActiveProfiles("test")
class UserServiceTest {
    @Autowired
    UserService userService;
}
```

```properties
# application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
```

---

# 11. How Auto-Configuration Works (Interview)

```text
@SpringBootApplication
  → @EnableAutoConfiguration
    → loads META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
      → each @Configuration class uses @ConditionalOn* to decide registration
```

Example: `DataSourceAutoConfiguration` only if JDBC on classpath and no user DataSource bean.

---

# Common Interview Questions

## Q1. @Profile vs @ConditionalOnProperty?

Profile = coarse environment switch. Property = fine-grained feature flag within environment.

## Q2. Can multiple profiles be active?

Yes: `spring.profiles.active=dev,debug`

## Q3. Default profile?

`default` profile if none specified.

## Q4. How does Spring Boot pick embedded Tomcat?

`@ConditionalOnClass(Servlet.class)` + `@ConditionalOnWebApplication`

## Q5. Order of condition evaluation?

`@AutoConfigureBefore/After`, `@Order` on condition — advanced auto-config ordering.

## Q6. @ConditionalOnMissingBean use case?

Let developers override auto-config beans with custom implementations.

---

# 12. HikariCP Connection Pool Tuning

Spring Boot default pool — **HikariCP**.

```properties
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.leak-detection-threshold=60000
```

| Property | Meaning |
|----------|---------|
| maximum-pool-size | Max connections (≈ cores × 2 for CPU-bound; higher for I/O-bound) |
| connection-timeout | Max wait for connection from pool (ms) |
| leak-detection-threshold | Log warning if connection held too long |

**Interview:** Pool too large → DB overload; too small → threads block waiting.

---

# One-Line Revision

```text
Profiles = environment-specific beans; @ConditionalOn* = register beans only when classpath/property/custom condition matches.
```

---

*End of Day 25 Spring Boot*
