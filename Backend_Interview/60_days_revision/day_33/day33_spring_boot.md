# Day 33 — @ConfigurationProperties

**Topics:** Type-Safe Config · Validation · Relaxed Binding · Interview Questions

---

# 1. Problem with @Value

```java
@Value("${payment.stripe.api-key}")
private String apiKey;

@Value("${payment.timeout:5000}")
private int timeout;
```

Issues: scattered keys, no grouping, no validation, typo-prone, hard to test.

---

# 2. @ConfigurationProperties

Bind prefix in `application.yml` to a POJO:

```yaml
app:
  payment:
    stripe-api-key: sk_live_xxx
    timeout-ms: 5000
    retry-max: 3
```

```java
@ConfigurationProperties(prefix = "app.payment")
@Validated
public class PaymentProperties {
    @NotBlank
    private String stripeApiKey;
    @Min(100)
    private int timeoutMs = 5000;
    @Max(10)
    private int retryMax = 3;

    // getters and setters
}
```

---

# 3. Enable Binding

```java
@Configuration
@EnableConfigurationProperties(PaymentProperties.class)
public class PaymentConfig { }

// Or register via @ConfigurationPropertiesScan on main class
@SpringBootApplication
@ConfigurationPropertiesScan
public class Application { }
```

```java
@Service
@RequiredArgsConstructor
public class PaymentService {
    private final PaymentProperties props;

    public void charge() {
        int timeout = props.getTimeoutMs();
    }
}
```

---

# 4. Relaxed Binding Rules

| Property file | Java field |
|---------------|------------|
| `stripe-api-key` | `stripeApiKey` |
| `stripe_api_key` | `stripeApiKey` |
| `STRIPE_API_KEY` | `stripeApiKey` |
| `timeout-ms` | `timeoutMs` |

Environment variables: `APP_PAYMENT_STRIPE_API_KEY` maps to `app.payment.stripe-api-key`.

---

# 5. Nested & List Properties

```yaml
app:
  servers:
    - name: primary
      url: https://api1.example.com
    - name: secondary
      url: https://api2.example.com
  database:
    host: localhost
    port: 3306
```

```java
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private List<Server> servers = new ArrayList<>();
    private Database database;

    public static class Server {
        private String name;
        private String url;
    }
    public static class Database {
        private String host;
        private int port;
    }
}
```

---

# 6. Validation

```java
@ConfigurationProperties(prefix = "app.payment")
@Validated
public class PaymentProperties {
    @NotBlank private String stripeApiKey;
    @Positive private int timeoutMs;
}
```

Fails fast at startup if config invalid (`@EnableConfigurationProperties` + `@Validated`).

---

# 7. @ConfigurationProperties vs @Value

| | @ConfigurationProperties | @Value |
|---|-------------------------|--------|
| Type-safe group | Yes | No |
| Validation | `@Validated` on class | Per-field possible |
| Relaxed binding | Yes | Limited |
| SpEL | No | Yes |
| Immutable | Constructor binding | Typically fields |

---

# 8. Constructor Binding (Immutable)

```java
@ConfigurationProperties(prefix = "app.payment")
public record PaymentProperties(
    @NotBlank String stripeApiKey,
    @Min(100) int timeoutMs
) {}
```

Spring Boot 3 + records: clean immutable config.

---

# 9. Secrets & Externalized Config

```text
Never commit secrets in application.yml
Use:
  - Environment variables
  - Spring Cloud Config + Vault
  - Kubernetes Secrets mounted as env
  - AWS Parameter Store / Secrets Manager
```

```yaml
app:
  payment:
    stripe-api-key: ${STRIPE_API_KEY}
```

---

# 10. Testing

```java
@SpringBootTest
@EnableConfigurationProperties(PaymentProperties.class)
@TestPropertySource(properties = {
    "app.payment.stripe-api-key=test-key",
    "app.payment.timeout-ms=1000"
})
class PaymentServiceTest {
    @Autowired PaymentProperties props;
}
```

Or `@DynamicPropertySource` for Testcontainers.

---

# Interview Questions

## @ConfigurationProperties vs @Configuration?

`@Configuration` defines beans. `@ConfigurationProperties` binds external config to a bean — often used together.

## How does Spring know to bind?

`@EnableConfigurationProperties` registers class as bean; `ConfigurationPropertiesBindingPostProcessor` binds Environment to fields.

## Multiple property sources precedence?

```text
Command line > OS env > application-{profile}.yml > application.yml
```

## Refresh without restart?

Spring Cloud `@RefreshScope` + `/actuator/refresh` for selected beans — not all properties hot-reloadable.

## Lombok with ConfigurationProperties?

`@ConstructorBinding` + `@RequiredArgsConstructor` or explicit getters/setters. `@Data` works but mutable.

---

# Quick Revision

```text
@ConfigurationProperties(prefix = "app.x")
@EnableConfigurationProperties / @ConfigurationPropertiesScan
Relaxed binding: kebab-case ↔ camelCase
@Validated for startup validation
Prefer over @Value for grouped config
Records for immutable binding
```

---

*End of Day 33 @ConfigurationProperties*
