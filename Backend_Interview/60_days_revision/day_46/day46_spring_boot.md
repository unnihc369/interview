# Day 46 — Spring Boot Annotations Review

**Topics:** Core · Web · JPA · Security · Testing · Config · Interview Q&A

---

# 1. Application Bootstrap

| Annotation | Purpose |
|------------|---------|
| `@SpringBootApplication` | `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| `@Configuration` | Java-based config, `@Bean` definitions |
| `@ComponentScan` | Scan package for `@Component` stereotypes |
| `@EnableAutoConfiguration` | Load auto-config based on classpath |
| `@Import` | Import additional config classes |
| `@Profile` | Activate beans by profile (`dev`, `prod`) |

```java
@SpringBootApplication
@EnableScheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

# 2. Stereotype Annotations (DI)

| Annotation | Layer |
|------------|-------|
| `@Component` | Generic Spring bean |
| `@Service` | Business logic |
| `@Repository` | Data access (exception translation) |
| `@Controller` | MVC controller (returns view) |
| `@RestController` | `@Controller` + `@ResponseBody` |

```java
@Service
@RequiredArgsConstructor  // Lombok constructor injection
public class OrderService {
    private final OrderRepository orderRepo;
    private final PaymentGateway paymentGateway;
}
```

---

# 3. Dependency Injection

| Annotation | Purpose |
|------------|---------|
| `@Autowired` | Inject by type (constructor preferred) |
| `@Qualifier` | Disambiguate multiple beans of same type |
| `@Primary` | Default bean when multiple candidates |
| `@Value` | Inject property `${app.name}` |
| `@Lazy` | Create bean on first use |

**Best practice:** Constructor injection (immutable, testable).

---

# 4. Web / REST Layer

| Annotation | Purpose |
|------------|---------|
| `@RequestMapping` | Base path + HTTP method mapping |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@PatchMapping` / `@DeleteMapping` | HTTP verbs |
| `@PathVariable` | URI template `{id}` |
| `@RequestParam` | Query param `?page=1` |
| `@RequestBody` | Deserialize JSON body |
| `@ResponseStatus` | Set HTTP status on method/class |
| `@Valid` | Trigger Bean Validation on DTO |
| `@ExceptionHandler` | Handle exceptions in controller/advice |

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public OrderResponse create(@Valid @RequestBody CreateOrderRequest req) {
        return orderService.create(req);
    }

    @GetMapping("/{id}")
    public OrderResponse get(@PathVariable Long id) {
        return orderService.getById(id);
    }
}
```

---

# 5. JPA / Data Layer

| Annotation | Purpose |
|------------|---------|
| `@Entity` | JPA entity |
| `@Table` | Table name, schema |
| `@Id` | Primary key |
| `@GeneratedValue` | Auto-generate ID |
| `@Column` | Column mapping |
| `@OneToMany` / `@ManyToOne` / `@ManyToMany` | Relationships |
| `@Transactional` | Transaction boundary |
| `@Query` | Custom JPQL/native SQL |
| `@Modifying` | UPDATE/DELETE queries |

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
}
```

---

# 6. Configuration & Properties

| Annotation | Purpose |
|------------|---------|
| `@ConfigurationProperties` | Type-safe property binding |
| `@EnableConfigurationProperties` | Register properties class |
| `@ConditionalOnProperty` | Bean if property matches |
| `@ConditionalOnClass` | Bean if class on classpath |
| `@ConditionalOnMissingBean` | Bean if not already defined |
| `@Bean` | Define bean in `@Configuration` |

```java
@ConfigurationProperties(prefix = "app.mail")
public record MailProperties(String host, int port, String from) {}

@Configuration
@EnableConfigurationProperties(MailProperties.class)
public class MailConfig {
    @Bean
    @ConditionalOnProperty(name = "app.mail.enabled", havingValue = "true")
    public JavaMailSender mailSender(MailProperties props) { ... }
}
```

---

# 7. Security

| Annotation | Purpose |
|------------|---------|
| `@EnableWebSecurity` | Enable Spring Security |
| `@PreAuthorize` | Method-level SpEL auth |
| `@Secured` | Role-based method security |
| `@EnableMethodSecurity` | Enable `@PreAuthorize` (Boot 3) |
| `@AuthenticationPrincipal` | Inject current user |

```java
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
public User getUser(@PathVariable Long userId) { ... }
```

---

# 8. Caching & Async

| Annotation | Purpose |
|------------|---------|
| `@EnableCaching` | Enable cache abstraction |
| `@Cacheable` | Cache method result |
| `@CacheEvict` | Remove cache entry |
| `@EnableAsync` | Enable async methods |
| `@Async` | Run method in thread pool |
| `@Scheduled` | Cron/fixed-rate tasks |

---

# 9. Testing Annotations

| Annotation | Purpose |
|------------|---------|
| `@SpringBootTest` | Full application context |
| `@WebMvcTest` | Controller slice |
| `@DataJpaTest` | JPA slice |
| `@MockBean` | Mock/replace Spring bean |
| `@AutoConfigureMockMvc` | Inject MockMvc |
| `@ActiveProfiles("test")` | Activate test profile |
| `@WithMockUser` | Mock authenticated user |

---

# 10. Validation

| Annotation | Purpose |
|------------|---------|
| `@NotNull` / `@NotBlank` / `@NotEmpty` | Required fields |
| `@Size` / `@Min` / `@Max` | Size/range |
| `@Email` / `@Pattern` | Format |
| `@Valid` | Cascade validation |

---

# 11. Quick Reference Map

```text
Bootstrapping:   @SpringBootApplication
DI:              @Service, @Repository, constructor injection
REST:            @RestController, @GetMapping, @PathVariable, @RequestBody
JPA:             @Entity, @ManyToOne, @Transactional
Config:          @ConfigurationProperties, @ConditionalOnProperty
Security:        @PreAuthorize, @EnableMethodSecurity
Testing:         @WebMvcTest, @MockBean, @SpringBootTest
```

---

# Interview Questions

## Q1. @Component vs @Service vs @Repository?

Functionally similar (all `@Component`). Semantic layers; `@Repository` adds exception translation.

## Q2. @Transactional on private method?

No effect — Spring AOP proxies only public methods on proxied beans.

## Q3. @RestController vs @Controller?

`@RestController` = `@Controller` + `@ResponseBody` on all methods — returns JSON not view name.

## Q4. @Autowired on field vs constructor?

Constructor: recommended (immutable, testable, required deps clear). Field: legacy, harder to test.

## Q5. Most important annotations for interview CRUD app?

`@SpringBootApplication`, `@RestController`, `@Service`, `@Repository`, `@Entity`, `@Transactional`, `@Valid`, `@SpringBootTest`, `@WebMvcTest`.

---

# One-Line Revision

```text
Spring Boot annotations = bootstrap (@SpringBootApplication) + stereotypes (@Service/@Repository) + web (@RestController) + JPA (@Entity/@Transactional) + config (@ConfigurationProperties) + test slices.
```

---

*End of Day 46 Spring Boot*
