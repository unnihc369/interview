# SDE 1–2 Supplement — Spring MVC, AOP & Cross-Cutting Concerns

**Covers gaps:** AOP · Filter vs Interceptor vs AOP · CORS · Multipart upload · Application Events · DTO mapping

**Related days:** Day 2 (MVC overview), Day 5 (@ControllerAdvice), Day 25 (auto-config), Day 37 (validation)

---

# 1. Spring MVC Request Lifecycle (Deep)

```text
Client HTTP Request
        ↓
┌───────────────────────────────────────┐
│  Servlet Filter Chain                 │
│  (Spring Security, CORS, logging)     │
└───────────────────────────────────────┘
        ↓
DispatcherServlet  ← front controller
        ↓
HandlerMapping     → finds @Controller + method for URL
        ↓
HandlerAdapter     → invokes method, binds args
        ↓
Controller method  → returns Object / ResponseEntity
        ↓
HttpMessageConverter (Jackson) → JSON/XML
        ↓
HTTP Response
```

## Key classes

| Class | Role |
|-------|------|
| `DispatcherServlet` | Routes all requests |
| `HandlerMapping` | URL → handler method |
| `HandlerAdapter` | Invokes handler with resolved args |
| `HttpMessageConverter` | Serializes/deserializes body |

---

# 2. Filter vs Interceptor vs AOP

| | Servlet Filter | HandlerInterceptor | Spring AOP |
|---|----------------|-------------------|------------|
| **Scope** | Servlet container | Spring MVC only | Any Spring bean method |
| **Runs before** | DispatcherServlet | Controller (after mapping) | Method call (proxy) |
| **Use for** | Security, encoding, CORS | Pre/post handle, auth check | Logging, transactions, metrics |
| **Registration** | `@Component` + `FilterRegistrationBean` | `WebMvcConfigurer.addInterceptors` | `@Aspect` + `@Around` |

```java
// Filter
@Component
public class RequestIdFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        MDC.put("requestId", UUID.randomUUID().toString());
        try { chain.doFilter(req, res); }
        finally { MDC.clear(); }
    }
}

// Interceptor
@Component
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        // return false to block request
        return true;
    }
}

// AOP
@Aspect
@Component
public class LoggingAspect {
    @Around("@annotation(Timed)")
    public Object logTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        log.info("{} took {}ms", pjp.getSignature(), System.currentTimeMillis() - start);
        return result;
    }
}
```

**Interview answer:** Filter is servlet-level (earliest). Interceptor is MVC-specific (after handler mapped). AOP is method-level via proxy (works on services too, not just controllers).

---

# 3. Spring AOP (Interview Critical)

## How proxies work

```text
Client → Proxy (JDK or CGLIB) → Target bean method
              ↓
         @Before / @Around advice runs here
```

| Proxy type | When |
|------------|------|
| **JDK dynamic proxy** | Target implements interface |
| **CGLIB proxy** | Concrete class, no interface |

**Self-invocation trap:** `this.save()` inside same class **bypasses proxy** — AOP won't run. Fix: inject self or split into another bean.

## Common annotations

```java
@Aspect
@Component
public class AuditAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void beforeService(JoinPoint jp) {
        log.info("Calling {}", jp.getSignature());
    }

    @AfterThrowing(pointcut = "execution(* com.example..*(..))", throwing = "ex")
    public void logError(JoinPoint jp, Exception ex) {
        log.error("Error in {}", jp.getSignature(), ex);
    }

    @Around("@annotation(org.springframework.transaction.annotation.Transactional)")
    public Object aroundTransactional(ProceedingJoinPoint pjp) throws Throwable {
        // custom transaction logging
        return pjp.proceed();
    }
}
```

| Annotation | When runs |
|------------|-----------|
| `@Before` | Before method |
| `@After` | After method (finally) |
| `@AfterReturning` | After normal return |
| `@AfterThrowing` | After exception |
| `@Around` | Wraps entire method — most powerful |

**Note:** Spring AOP is **proxy-based**, not full AspectJ weaving — only **public** methods on Spring beans are advised.

---

# 4. CORS Configuration

Browser blocks cross-origin requests unless server allows.

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://myapp.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

**Spring Security:** CORS must be configured **before** or **with** Security filter chain:

```java
http.cors(cors -> cors.configurationSource(request -> {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://myapp.com"));
    return config;
}));
```

**Preflight:** Browser sends OPTIONS request first — must return 200 with CORS headers.

---

# 5. Multipart File Upload

```java
@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public ResponseEntity<FileResponse> upload(@RequestParam("file") MultipartFile file) {
    if (file.isEmpty()) throw new BadRequestException("Empty file");
    if (file.getSize() > MAX_SIZE) throw new BadRequestException("Too large");
    String path = storageService.save(file.getOriginalFilename(), file.getInputStream());
    return ResponseEntity.ok(new FileResponse(path));
}
```

```properties
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

---

# 6. Spring Application Events

Decouple components — publisher doesn't know subscribers.

```java
// Event
public record OrderPlacedEvent(Long orderId, String userId) { }

// Publisher
@Service
@RequiredArgsConstructor
public class OrderService {
    private final ApplicationEventPublisher events;

    public void placeOrder(OrderRequest req) {
        Order order = save(req);
        events.publishEvent(new OrderPlacedEvent(order.getId(), req.userId()));
    }
}

// Listener
@Component
public class EmailListener {
    @EventListener
    @Async
    public void onOrderPlaced(OrderPlacedEvent event) {
        emailService.sendConfirmation(event.orderId());
    }
}
```

**vs Observer pattern:** Spring Events is application-level Observer with async support via `@Async`.

---

# 7. DTO ↔ Entity Best Practices

```text
Controller → DTO (request/response)
Service    → Entity (domain/DB)
Repository → Entity
```

```java
// Manual mapping
public UserResponse toResponse(User user) {
    return new UserResponse(user.getId(), user.getEmail(), user.getName());
}

// MapStruct (compile-time, no reflection)
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserResponse toResponse(User user);
    User toEntity(UserRequest dto);
}
```

**Never return Entity from REST** — exposes schema, triggers lazy load in JSON serializer, breaks API contract on schema change.

---

# Interview Questions

## Q1. Filter vs Interceptor?

Filter runs at servlet level before DispatcherServlet. Interceptor runs after handler mapped, MVC-specific.

## Q2. Why AOP doesn't work on private methods?

Spring AOP uses proxies — only public methods on proxied beans are intercepted.

## Q3. JDK proxy vs CGLIB?

JDK needs interface. CGLIB subclasses concrete class (final methods can't be proxied).

## Q4. Self-invocation AOP problem?

Calling method on `this` bypasses proxy — inject self or extract to another `@Service`.

## Q5. Where to configure CORS with Spring Security?

In SecurityFilterChain via `http.cors()` — Security filter runs before MVC.

---

# One-Line Revision

```text
Filter (servlet) → Interceptor (MVC) → AOP (any bean method via proxy).
CORS = browser cross-origin headers; configure with Security.
DTO in API, Entity in persistence; MapStruct for mapping.
```

---

*End of Spring MVC & AOP Supplement*
