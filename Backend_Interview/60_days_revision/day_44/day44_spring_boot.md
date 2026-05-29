# Day 44 — @ConditionalOnProperty & Custom Auto-Configuration

**Topics:** Conditional beans · Starter modules · `@ConfigurationProperties` · Interview Q&A

---

# 1. Spring Boot Auto-Configuration Recap

Spring Boot scans `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 3) and conditionally loads beans based on classpath, properties, existing beans.

```text
@Configuration + @Conditional* + @EnableConfigurationProperties
= Custom auto-config module
```

---

# 2. @ConditionalOnProperty

Activates bean(s) only when property matches.

```java
@Configuration
@ConditionalOnProperty(
    name = "notification.email.enabled",
    havingValue = "true",
    matchIfMissing = false
)
public class EmailNotificationConfig {

    @Bean
    public EmailSender emailSender(EmailProperties props) {
        return new SmtpEmailSender(props.getHost(), props.getPort());
    }
}
```

```yaml
notification:
  email:
    enabled: true
    host: smtp.example.com
    port: 587
```

---

# Common Variations

```java
// Prefix match — any property under prefix
@ConditionalOnProperty(prefix = "cache.redis", name = "enabled", havingValue = "true")

// matchIfMissing = true → enabled by default
@ConditionalOnProperty(name = "feature.x", havingValue = "true", matchIfMissing = true)

// List of required properties
@ConditionalOnProperty(name = {"a", "b"})  // both must be defined (not false)
```

---

# 3. Other Conditional Annotations

| Annotation | Condition |
|------------|-----------|
| `@ConditionalOnClass` | Class on classpath |
| `@ConditionalOnMissingBean` | Bean not already defined |
| `@ConditionalOnBean` | Bean exists |
| `@ConditionalOnWebApplication` | Web app context |
| `@ConditionalOnResource` | Resource exists on classpath |

```java
@Configuration
@ConditionalOnClass(RedisTemplate.class)
@ConditionalOnProperty(name = "cache.type", havingValue = "redis")
public class RedisCacheAutoConfig {

    @Bean
    @ConditionalOnMissingBean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        return RedisCacheManager.builder(factory).build();
    }
}
```

---

# 4. Custom Auto-Configuration — Full Example

## Step 1: Properties Class

```java
@ConfigurationProperties(prefix = "payment.stripe")
public record StripeProperties(
    boolean enabled,
    String apiKey,
    String webhookSecret
) {}
```

## Step 2: Auto-Configuration Class

```java
@Configuration
@EnableConfigurationProperties(StripeProperties.class)
@ConditionalOnProperty(prefix = "payment.stripe", name = "enabled", havingValue = "true")
@ConditionalOnClass(name = "com.stripe.Stripe")
public class StripeAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public PaymentGateway stripePaymentGateway(StripeProperties props) {
        Stripe.apiKey = props.apiKey();
        return new StripePaymentGateway();
    }

    @Bean
    @ConditionalOnMissingBean
    public StripeWebhookHandler stripeWebhookHandler(StripeProperties props) {
        return new StripeWebhookHandler(props.webhookSecret());
    }
}
```

## Step 3: Register Auto-Config

`src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`:

```text
com.mycompany.payment.StripeAutoConfiguration
```

(Boot 2.x used `META-INF/spring.factories` with `EnableAutoConfiguration` key.)

---

# 5. Custom Starter Module Structure

```text
payment-spring-boot-starter/
├── pom.xml
├── PaymentAutoConfiguration.java
├── PaymentProperties.java
├── PaymentGateway.java (interface)
├── StripePaymentGateway.java
└── META-INF/spring/
    └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

Consumer `pom.xml`:

```xml
<dependency>
    <groupId>com.mycompany</groupId>
    <artifactId>payment-spring-boot-starter</artifactId>
</dependency>
```

Consumer `application.yml`:

```yaml
payment:
  stripe:
    enabled: true
    api-key: ${STRIPE_API_KEY}
```

---

# 6. @AutoConfigureBefore / @AutoConfigureAfter

Control ordering when configs depend on each other.

```java
@Configuration
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
@ConditionalOnProperty(name = "audit.enabled", havingValue = "true")
public class AuditAutoConfiguration {

    @Bean
    public AuditInterceptor auditInterceptor(DataSource dataSource) {
        return new AuditInterceptor(dataSource);
    }
}
```

---

# 7. Testing Conditional Config

```java
@SpringBootTest
@TestPropertySource(properties = {
    "payment.stripe.enabled=true",
    "payment.stripe.api-key=test-key"
})
class StripeAutoConfigTest {

    @Autowired
    PaymentGateway gateway;

    @Test
    void gatewayBeanLoaded() {
        assertThat(gateway).isInstanceOf(StripePaymentGateway.class);
    }
}

@SpringBootTest
@TestPropertySource(properties = "payment.stripe.enabled=false")
class StripeDisabledTest {

    @Autowired(required = false)
    PaymentGateway gateway;

    @Test
    void gatewayNotLoaded() {
        assertThat(gateway).isNull();
    }
}
```

---

# Interview Questions

## Q1. @ConditionalOnProperty vs @Profile?

`@Profile("prod")` activates by active profile name. `@ConditionalOnProperty` activates by any property value — finer-grained feature flags.

## Q2. Why @ConditionalOnMissingBean?

Allows users to override auto-config beans with their own `@Bean` — standard Spring Boot extensibility pattern.

## Q3. How does Spring Boot know which auto-configs to load?

`AutoConfiguration.imports` file + `@SpringBootApplication` triggers `@EnableAutoConfiguration`.

## Q4. Property binding: @Value vs @ConfigurationProperties?

`@Value` — single property, SpEL. `@ConfigurationProperties` — type-safe binding of prefix, validation, IDE metadata — preferred for config classes.

## Q5. Create internal starter for microservices?

Extract shared security, logging, tracing into `company-spring-boot-starter` with conditional modules — DRY across 20+ services.

---

# One-Line Revision

```text
Custom auto-config = @Configuration + @ConditionalOn* + @EnableConfigurationProperties + META-INF AutoConfiguration.imports; @ConditionalOnProperty = feature flags.
```

---

*End of Day 44 Spring Boot*
