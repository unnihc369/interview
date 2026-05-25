# Spring Boot Annotations — Complete Interview Guide

## Table of Contents

1. [Java Annotations Fundamentals](#1-java-annotations-fundamentals)
2. [Reflection in Java](#2-reflection-in-java)
3. [How Spring Uses Annotations & Reflection](#3-how-spring-uses-annotations--reflection)
4. [Before Annotations — XML Configuration Era](#4-before-annotations--xml-configuration-era)
5. [Meta-Annotations & Composed Annotations](#5-meta-annotations--composed-annotations)
6. [@SpringBootApplication](#6-springbootapplication)
7. [Web Layer — @Controller & @RestController](#7-web-layer--controller--restcontroller)
8. [Request Mapping Annotations](#8-request-mapping-annotations)
9. [@RequestParam, @PathVariable, @RequestBody](#9-requestparam-pathvariable-requestbody)
10. [@ResponseBody & ResponseEntity](#10-responsebody--responseentity)
11. [Request Parameter Type Conversion](#11-request-parameter-type-conversion)
12. [@InitBinder & PropertyEditor](#12-initbinder--propertyeditor)
13. [JSON Serialization — Jackson vs Gson](#13-json-serialization--jackson-vs-gson)
14. [Dependency Injection — @Autowired](#14-dependency-injection--autowired)
15. [Injection Styles — Constructor, Setter, Field](#15-injection-styles--constructor-setter-field)
16. [@Qualifier & Bean Naming](#16-qualifier--bean-naming)
17. [Bean Scopes — @Scope](#17-bean-scopes--scope)
18. [Stereotype Annotations — @Component, @Service, @Repository](#18-stereotype-annotations--component-service-repository)
19. [Configuration Annotations](#19-configuration-annotations)
20. [JPA / Data Annotations (Quick Reference)](#20-jpa--data-annotations-quick-reference)
21. [Final Mental Model](#21-final-mental-model)
22. [Interview Questions Index](#22-interview-questions-index)

---

## 1. Java Annotations Fundamentals

### What is an Annotation?

An **annotation** is metadata added to Java code (classes, methods, fields, parameters) that does **nothing by itself** at runtime unless something **reads** it (compiler, framework, or your code via Reflection).

```java
@Override
public String toString() { return "hello"; }
```

`@Override` tells the **compiler** to check that you are actually overriding a parent method.

### Built-in Java Annotations

| Annotation | Target | Purpose |
|------------|--------|---------|
| `@Override` | Method | Compiler checks override |
| `@Deprecated` | Any | Marks as obsolete |
| `@SuppressWarnings` | Any | Suppress compiler warnings |
| `@FunctionalInterface` | Interface | Single abstract method |
| `@SafeVarargs` | Method/Constructor | Varargs safety |

### Custom Annotation Example

```java
// Define annotation
@Retention(RetentionPolicy.RUNTIME)   // available at runtime via Reflection
@Target(ElementType.METHOD)           // can only be used on methods
public @interface LogExecutionTime {
    String value() default "";
}

// Use annotation
public class PaymentService {
    @LogExecutionTime("processPayment")
    public void processPayment() {
        // business logic
    }
}
```

### Retention Policies

| Policy | When available |
|--------|----------------|
| `SOURCE` | Compile time only (e.g. `@Override`) — discarded by compiler |
| `CLASS` | In `.class` file — not at runtime by default |
| `RUNTIME` | Available at runtime — **Spring uses this** |

### Target (Where annotation can be placed)

```java
@Target({
    ElementType.TYPE,       // class, interface, enum
    ElementType.METHOD,
    ElementType.FIELD,
    ElementType.PARAMETER,
    ElementType.CONSTRUCTOR
})
```

> **Interview line:** Spring annotations use `RUNTIME` retention so the framework can read them when the application starts and wire beans automatically.

---

## 2. Reflection in Java

### What is Reflection?

**Reflection** allows Java code to **inspect and modify** classes, methods, fields, and annotations **at runtime** — even without knowing the class name at compile time.

```java
public class ReflectionDemo {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("com.example.UserService");

        // Get all methods
        for (Method method : clazz.getDeclaredMethods()) {
            System.out.println("Method: " + method.getName());

            // Check if method has custom annotation
            if (method.isAnnotationPresent(LogExecutionTime.class)) {
                LogExecutionTime ann = method.getAnnotation(LogExecutionTime.class);
                System.out.println("  → Has @LogExecutionTime: " + ann.value());
            }
        }

        // Create instance without 'new' keyword
        Object instance = clazz.getDeclaredConstructor().newInstance();

        // Invoke method by name
        Method m = clazz.getMethod("getUser", int.class);
        Object result = m.invoke(instance, 42);
    }
}
```

### Common Reflection APIs

| API | Purpose |
|-----|---------|
| `Class.forName("...")` | Load class by name |
| `getDeclaredMethods()` | All methods (including private) |
| `getAnnotation(SomeAnn.class)` | Read annotation on element |
| `newInstance()` / `getDeclaredConstructor().newInstance()` | Create object |
| `method.invoke(obj, args)` | Call method dynamically |
| `field.set(obj, value)` | Set field value |

### Why Reflection Matters for Spring

Spring **does not** hardcode your classes. At startup it:

1. Scans classpath for classes with `@Component`, `@Controller`, etc.
2. Uses **Reflection** to read annotation metadata
3. Creates bean instances and injects dependencies
4. Maps `@GetMapping` URLs to methods via Reflection

```
Application starts
    ↓
Component scan finds UserController.class
    ↓
Reflection reads @RestController, @RequestMapping("/users")
    ↓
Reflection finds method with @GetMapping("/{id}")
    ↓
DispatcherServlet registers handler mapping
```

---

## 3. How Spring Uses Annotations & Reflection

| Step | What Spring does |
|------|------------------|
| 1 | `@ComponentScan` scans packages |
| 2 | Finds classes with stereotype annotations |
| 3 | Reflection creates bean definitions |
| 4 | Resolves `@Autowired` dependencies |
| 5 | Registers `@RequestMapping` handlers |
| 6 | Wraps beans in proxies (for `@Transactional`, etc.) |

**Without Reflection**, you would need XML or manual registration for every class and every endpoint.

---

## 4. Before Annotations — XML Configuration Era

### Old Spring MVC (no annotations)

**1. Register beans in XML:**

```xml
<!-- applicationContext.xml -->
<beans>
    <bean id="userService" class="com.example.UserService"/>
    <bean id="userController" class="com.example.UserController">
        <property name="userService" ref="userService"/>
    </bean>
</beans>
```

**2. Configure DispatcherServlet + URL mapping in XML:**

```xml
<!-- spring-mvc.xml -->
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
<bean id="urlMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <prop key="/users">userController</prop>
        </props>
    </property>
</bean>
```

**3. Controller implemented interface (no `@GetMapping`):**

```java
public class UserController implements Controller {
  @Override
  public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse res) {
    // manual request parsing
    return new ModelAndView("userView");
  }
}
```

### Annotation era (modern)

```java
@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

| Old (XML) | Modern (Annotations) |
|-----------|------------------------|
| `web.xml` servlet mapping | `@RequestMapping` |
| XML bean definitions | `@Component`, `@Service` |
| XML property injection | `@Autowired` |
| `Controller` interface | `@Controller` / `@RestController` |
| Manual JSON parsing | Jackson + `@RequestBody` |

---

## 5. Meta-Annotations & Composed Annotations

A **meta-annotation** is an annotation applied to another annotation.

A **composed annotation** combines multiple annotations into one convenient annotation.

### Example — How `@RestController` is built

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller                    // ← meta: marks as Spring MVC controller
@ResponseBody                  // ← meta: return value → HTTP body (JSON)
public @interface RestController {
    @AliasFor(annotation = Controller.class)
    String value() default "";
}
```

So when you write `@RestController`, Spring sees **both** `@Controller` and `@ResponseBody`.

### Composed Annotations Cheat Sheet

| Annotation | Composed / includes |
|------------|---------------------|
| `@SpringBootApplication` | `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| `@RestController` | `@Controller` + `@ResponseBody` |
| `@GetMapping` | `@RequestMapping(method = GET)` |
| `@PostMapping` | `@RequestMapping(method = POST)` |
| `@PutMapping` | `@RequestMapping(method = PUT)` |
| `@DeleteMapping` | `@RequestMapping(method = DELETE)` |
| `@Service` | `@Component` (specialized stereotype) |
| `@Repository` | `@Component` + exception translation |
| `@Controller` | `@Component` (specialized stereotype) |

### @SpringBootApplication breakdown

```java
@SpringBootApplication
// Equivalent to:
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application { }
```

| Part | Purpose |
|------|---------|
| `@Configuration` | Class can define `@Bean` methods |
| `@EnableAutoConfiguration` | Auto-configure Spring Boot starters |
| `@ComponentScan` | Scan current package and sub-packages for beans |

---

## 6. @SpringBootApplication

### Purpose

Single entry-point annotation that bootstraps a Spring Boot application.

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### What happens at startup

```
@SpringBootApplication detected
    ↓
@ComponentScan → find @Service, @Repository, @RestController
    ↓
@EnableAutoConfiguration → configure DataSource, Tomcat, Jackson, etc.
    ↓
Embedded Tomcat starts on port 8080
```

### Before Spring Boot

You needed separate classes: `AppConfig`, `DispatcherServlet` initializer, `web.xml` — all replaced by this one annotation.

---

## 7. Web Layer — @Controller & @RestController

### @Controller

Marks class as a **Spring MVC controller**. Method return values are typically **view names** (Thymeleaf, JSP).

```java
@Controller
@RequestMapping("/pages")
public class PageController {

    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("title", "Home");
        return "home";   // resolves to templates/home.html
    }
}
```

**Is it composed?** Yes — `@Controller` is meta-annotated with `@Component`.

### @RestController

For **REST APIs** — return value goes directly to HTTP response body (usually JSON).

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return new User(id, "Alice");
    }
}
```

**Is it composed?** Yes — `@Controller` + `@ResponseBody`.

| @Controller | @RestController |
|-------------|-----------------|
| Returns view name | Returns data (JSON/XML) |
| Server-side rendering | REST API |
| Needs `@ResponseBody` per method for JSON | JSON by default on all methods |

---

## 8. Request Mapping Annotations

### @RequestMapping

Maps HTTP requests to handler methods. Can be used on **class** and **method** level.

```java
@RestController
@RequestMapping("/api/products")   // base path
public class ProductController {

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public Product get(@PathVariable Long id) { ... }

    @RequestMapping(value = "", method = RequestMethod.POST)
    public Product create(@RequestBody Product product) { ... }
}
```

### HTTP method shortcuts (composed annotations)

| Annotation | Equivalent |
|------------|------------|
| `@GetMapping("/path")` | `@RequestMapping(value="/path", method=GET)` |
| `@PostMapping("/path")` | `@RequestMapping(value="/path", method=POST)` |
| `@PutMapping("/path")` | `@RequestMapping(value="/path", method=PUT)` |
| `@DeleteMapping("/path")` | `@RequestMapping(value="/path", method=DELETE)` |
| `@PatchMapping("/path")` | `@RequestMapping(value="/path", method=PATCH)` |

### Full CRUD example

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping
    public List<Order> getAll() { ... }

    @GetMapping("/{id}")
    public Order getById(@PathVariable Long id) { ... }

    @PostMapping
    public Order create(@RequestBody OrderRequest request) { ... }

    @PutMapping("/{id}")
    public Order update(@PathVariable Long id, @RequestBody OrderRequest request) { ... }

    @DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) { ... }
}
```

### Before annotations

URL → controller mapping was in **XML** (`SimpleUrlHandlerMapping`) or `web.xml`, not on the Java method.

---

## 9. @RequestParam, @PathVariable, @RequestBody

### @PathVariable — values from URL path

```
GET /api/users/42/profile
              ↑
         path variable
```

```java
@GetMapping("/users/{id}/profile")
public Profile getProfile(@PathVariable("id") Long userId) {
    return profileService.get(userId);
}

// Shorthand when name matches
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) { ... }
```

### @RequestParam — values from query string

```
GET /api/users?page=1&size=10&sort=name
```

```java
@GetMapping("/users")
public List<User> search(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(required = false) String sort
) {
    return userService.search(page, size, sort);
}

// Optional param
@GetMapping("/search")
public List<User> search(@RequestParam(required = false) String name) { ... }
```

### @RequestBody — JSON body → Java object

```java
@PostMapping("/users")
public User create(@RequestBody UserRequest request) {
    return userService.save(request);
}
```

**Request:**
```http
POST /api/users
Content-Type: application/json

{"name": "Alice", "email": "alice@example.com"}
```

Spring uses **Jackson** (by default) to deserialize JSON → `UserRequest`.

### Comparison

| Annotation | Source | Example |
|------------|--------|---------|
| `@PathVariable` | URL path | `/users/{id}` |
| `@RequestParam` | Query string | `?page=1&size=10` |
| `@RequestBody` | HTTP body (JSON) | `{"name":"Alice"}` |
| `@RequestHeader` | HTTP header | `Authorization: Bearer ...` |
| `@CookieValue` | Cookie | `sessionId=abc` |

---

## 10. @ResponseBody & ResponseEntity

### @ResponseBody

Tells Spring: serialize return value directly to HTTP response body (not a view name).

```java
@Controller
public class MixedController {

    @GetMapping("/data")
    @ResponseBody   // required here because class is @Controller, not @RestController
    public Map<String, String> getData() {
        return Map.of("status", "ok");
    }
}
```

On `@RestController`, `@ResponseBody` is **implicit** on every method.

### ResponseEntity — full control over HTTP response

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(user -> ResponseEntity.ok(user))
        .orElse(ResponseEntity.notFound().build());
}

@PostMapping("/users")
public ResponseEntity<User> create(@RequestBody UserRequest req) {
    User saved = userService.save(req);
    return ResponseEntity
        .status(HttpStatus.CREATED)
        .header("Location", "/users/" + saved.getId())
        .body(saved);
}
```

| Return type | Use when |
|-------------|----------|
| `User` (on `@RestController`) | Simple 200 + JSON body |
| `ResponseEntity<User>` | Custom status, headers, optional body |
| `String` (view name on `@Controller`) | Server-rendered HTML |

**Note:** `ResponseEntity` is a **class**, not an annotation. Often listed alongside web annotations in interviews.

---

## 11. Request Parameter Type Conversion

The framework automatically converts the **string** from the request into the **Java type** you declare.

### Supported types (automatic conversion)

| Category | Examples |
|----------|----------|
| Primitives | `int`, `long`, `float`, `double`, `boolean` |
| Wrapper classes | `Integer`, `Long`, `Float`, `Double`, `Boolean` |
| String | Always available |
| Enums | `?status=ACTIVE` → enum constant |
| Dates (with format) | `@DateTimeFormat` |
| Collections | `?ids=1,2,3` → `List<Integer>` |

```java
@GetMapping("/orders")
public List<Order> filter(
    @RequestParam OrderStatus status,      // ACTIVE, PENDING, etc.
    @RequestParam int page,
    @RequestParam boolean active
) { ... }

public enum OrderStatus { ACTIVE, PENDING, CANCELLED }
```

### Conversion flow

```
HTTP request: ?page=2&active=true
        ↓
DispatcherServlet
        ↓
ConversionService / PropertyEditor
        ↓
Method parameter: int page=2, boolean active=true
```

---

## 12. @InitBinder & PropertyEditor

### When default conversion is not enough

For **custom objects** or special formats, register converters via `@InitBinder`.

### PropertyEditor (classic Spring approach)

```java
// Custom type
public class Money {
    private final BigDecimal amount;
    public Money(BigDecimal amount) { this.amount = amount; }
    public BigDecimal getAmount() { return amount; }
}

// PropertyEditor: String → Money
public class MoneyPropertyEditor extends PropertyEditorSupport {
    @Override
    public void setAsText(String text) {
        setValue(new Money(new BigDecimal(text)));
    }
}
```

### @InitBinder — register editor for this controller

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(Money.class, new MoneyPropertyEditor());
    }

    @GetMapping
    public String pay(@RequestParam Money amount) {
        // ?amount=99.50  →  Money(99.50)
        return "Paid: " + amount.getAmount();
    }
}
```

### Modern alternative — Converter (preferred today)

```java
@Component
public class StringToMoneyConverter implements Converter<String, Money> {
    @Override
    public Money convert(String source) {
        return new Money(new BigDecimal(source));
    }
}
// Register via WebMvcConfigurer — thread-safe, no @InitBinder needed
```

| Approach | Thread-safe? | Typical use |
|----------|--------------|-------------|
| `PropertyEditor` | No (per-request via `@InitBinder`) | Legacy, custom binding |
| `Converter` / `Formatter` | Yes | Modern Spring apps |
| Jackson (`@RequestBody`) | Yes | JSON APIs |

### @ModelAttribute — bind multiple params to object

```java
@GetMapping("/search")
public List<User> search(@ModelAttribute UserSearchCriteria criteria) {
    // ?name=Alice&minAge=18  →  UserSearchCriteria object
    return userService.search(criteria);
}

public class UserSearchCriteria {
    private String name;
    private int minAge;
    // getters/setters
}
```

---

## 13. JSON Serialization — Jackson vs Gson

Spring Boot uses **Jackson** by default for `@RequestBody` and `@ResponseBody`. **Gson** is optional (not default).

### Jackson (default in Spring Boot)

**Dependency:** `spring-boot-starter-web` includes `jackson-databind`.

```java
// Automatic with @RestController — no extra config needed
@PostMapping("/users")
public User create(@RequestBody UserRequest request) {
    return userService.save(request);
}
```

**Jackson annotations:**

```java
public class User {
    @JsonProperty("user_name")          // custom JSON field name
    private String name;

    @JsonIgnore
    private String password;

    @JsonFormat(pattern = "yyyy-MM-dd")
    private LocalDate birthDate;
}
```

**Manual Jackson usage:**

```java
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(user);
User parsed = mapper.readValue(json, User.class);
```

### Gson (Google — not Spring default)

**Dependency:** add manually:

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
</dependency>
```

```java
Gson gson = new Gson();
String json = gson.toJson(user);
User user = gson.fromJson(json, User.class);
```

**Gson annotations:**

```java
public class User {
    @SerializedName("user_name")
    private String name;

    @Expose(serialize = false)
    private String password;
}
```

### Jackson vs Gson

| Feature | Jackson | Gson |
|---------|---------|------|
| Spring Boot default | Yes | No |
| Performance | Faster (generally) | Good |
| Annotations | `@JsonProperty`, `@JsonIgnore` | `@SerializedName`, `@Expose` |
| Java 8 date/time | Built-in modules | Needs adapters |
| Spring MVC integration | Native (`@RequestBody`) | Manual / custom config |
| Use case | Spring REST APIs | Android, standalone JSON |

### Replacing Jackson with Gson in Spring (rare)

Requires custom `HttpMessageConverter` — **not recommended** unless you have a strong reason. Stick with Jackson in Spring Boot.

---

## 14. Dependency Injection — @Autowired

### Purpose

Tells Spring IoC container to **inject** a matching bean automatically.

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;   // Spring injects bean

    public void checkout() {
        paymentService.charge();
    }
}
```

### How Spring finds the bean

```
@Autowired PaymentService
        ↓
Search IoC container for bean of type PaymentService
        ↓
If exactly one → inject
If zero → error (unless optional)
If multiple → need @Qualifier or @Primary
```

### @Autowired is NOT a composed annotation

It is a single annotation from `org.springframework.beans.factory.annotation`.

### Optional injection

```java
@Autowired(required = false)
private EmailService emailService;   // null if bean not found
```

---

## 15. Injection Styles — Constructor, Setter, Field

### 1. Constructor Injection (recommended)

```java
@Service
public class OrderService {

    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    @Autowired   // optional in Spring 4.3+ if only one constructor
    public OrderService(PaymentService paymentService,
                        InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
}
```

**Lombok shortcut:**

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final PaymentService paymentService;   // injected via constructor
}
```

| Advantage | Why |
|-----------|-----|
| Immutable dependencies | `final` fields |
| Easy unit testing | Pass mocks in constructor |
| Required dependencies clear | Cannot create bean without deps |
| Spring recommended | Best practice |

### 2. Setter Injection

```java
@Service
public class OrderService {
    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Use for **optional** dependencies.

### 3. Field Injection

```java
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;
}
```

| Style | Recommended? |
|-------|--------------|
| Constructor | Yes |
| Setter | Optional deps only |
| Field | Avoid (harder to test, hides dependencies) |

### Before annotations (XML era)

```xml
<bean id="orderService" class="com.example.OrderService">
    <constructor-arg ref="paymentService"/>
</bean>
```

---

## 16. @Qualifier & Bean Naming

### Problem — multiple beans of same type

```java
@Service("emailNotifier")
public class EmailNotificationService implements NotificationService { }

@Service("smsNotifier")
public class SmsNotificationService implements NotificationService { }
```

### Solution — @Qualifier

```java
@Service
public class OrderService {

    private final NotificationService notificationService;

    @Autowired
    public OrderService(@Qualifier("smsNotifier") NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}
```

### @Primary — default bean when multiple exist

```java
@Service
@Primary
public class EmailNotificationService implements NotificationService { }

@Service
public class SmsNotificationService implements NotificationService { }

@Service
public class OrderService {
    @Autowired
    private NotificationService notificationService;  // gets Email (Primary)
}
```

### Bean name

```java
@Component("myCustomBeanName")
public class MyService { }

// Inject by name
@Autowired
@Qualifier("myCustomBeanName")
private MyService myService;
```

| Mechanism | Purpose |
|-----------|---------|
| `@Qualifier("name")` | Pick specific bean by name |
| `@Primary` | Default when multiple candidates |
| `@Component("name")` | Explicit bean name |

---

## 17. Bean Scopes — @Scope

Controls **how many instances** Spring creates for a bean.

| Scope | Annotation | Instances | Use case |
|-------|------------|-----------|----------|
| **Singleton** | `@Scope("singleton")` or default | One per Spring container | Services, repositories |
| **Prototype** | `@Scope("prototype")` | New instance every injection | Stateful objects |
| **Request** | `@Scope("request")` | One per HTTP request | Web apps |
| **Session** | `@Scope("session")` | One per HTTP session | User cart, login state |
| **Application** | `@Scope("application")` | One per ServletContext | Web apps |

```java
@Service
@Scope("singleton")   // default — can omit
public class UserService { }

@Component
@Scope("prototype")
public class RequestProcessor { }

@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedBean { }
```

### Singleton vs Prototype

```java
// Singleton — same object everywhere
UserService a = context.getBean(UserService.class);
UserService b = context.getBean(UserService.class);
a == b;  // true

// Prototype — new object each time
RequestProcessor x = context.getBean(RequestProcessor.class);
RequestProcessor y = context.getBean(RequestProcessor.class);
x == y;  // false
```

### Interview note

Default scope for Spring beans is **singleton**. Do not confuse with Gang of Four Singleton pattern — Spring manages one instance **per container**.

---

## 18. Stereotype Annotations — @Component, @Service, @Repository

All register the class as a **Spring bean** (via `@Component` meta-annotation).

| Annotation | Layer | Extra behavior |
|------------|-------|----------------|
| `@Component` | Generic | Basic bean |
| `@Service` | Business logic | Semantic only (= `@Component`) |
| `@Repository` | Data access | Exception translation (`SQLException` → Spring `DataAccessException`) |
| `@Controller` | Web (MVC) | MVC controller |
| `@RestController` | REST API | `@Controller` + `@ResponseBody` |

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> { }

@Service
public class UserService {
    private final UserRepository userRepository;
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

@RestController
public class UserController {
    private final UserService userService;
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

---

## 19. Configuration Annotations

| Annotation | Purpose |
|------------|---------|
| `@Configuration` | Class defines `@Bean` methods |
| `@Bean` | Method produces a bean managed by Spring |
| `@Value` | Inject property from `application.properties` |
| `@PropertySource` | Load custom properties file |
| `@Profile` | Bean active only for specific profile (`dev`, `prod`) |

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder().build();
    }
}

@Service
public class AppService {
    @Value("${app.name}")
    private String appName;
}
```

---

## 20. JPA / Data Annotations (Quick Reference)

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Class maps to database table |
| `@Table(name = "users")` | Table name |
| `@Id` | Primary key |
| `@GeneratedValue` | Auto-generate ID |
| `@Column` | Column mapping |
| `@OneToMany`, `@ManyToOne` | Relationships |
| `@Transactional` | Transaction boundary (service layer) |

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;
}
```

---

## 21. Final Mental Model

```
Annotations (metadata on code)
        ↓
Reflection (Spring reads at runtime)
        ↓
IoC Container (creates & wires beans)
        ↓
DispatcherServlet (maps HTTP → @GetMapping methods)
        ↓
Jackson (JSON ↔ Java objects)
        ↓
HTTP Response
```

| Concept | Role |
|---------|------|
| `@Component` family | Register beans |
| `@Autowired` | Wire dependencies |
| `@Qualifier` / `@Primary` | Resolve conflicts |
| `@Scope` | Instance lifecycle |
| `@RestController` + mappings | REST endpoints |
| `@RequestBody` / `@ResponseBody` | JSON body |
| `@InitBinder` | Custom request binding |
| Jackson | Default JSON library |

### One-Line Interview Summary

> Spring annotations are runtime-retained metadata that Spring reads via Reflection to auto-configure beans, map HTTP requests, inject dependencies, and serialize JSON — replacing the older XML-based configuration model.

---

## 22. Interview Questions Index

### Basics

| # | Question | Answer in section |
|---|----------|-------------------|
| Q1 | [What is a Java annotation?](#1-java-annotations-fundamentals) | [§1 Java Annotations](#1-java-annotations-fundamentals) |
| Q2 | [What is Reflection?](#2-reflection-in-java) | [§2 Reflection](#2-reflection-in-java) |
| Q3 | [How does Spring use Reflection?](#3-how-spring-uses-annotations--reflection) | [§3 Spring + Reflection](#3-how-spring-uses-annotations--reflection) |
| Q4 | [What is @SpringBootApplication?](#6-springbootapplication) | [§6 @SpringBootApplication](#6-springbootapplication) |
| Q5 | [Difference between @Controller and @RestController?](#7-web-layer--controller--restcontroller) | [§7 Controller vs RestController](#7-web-layer--controller--restcontroller) |
| Q6 | [What is @RequestMapping?](#8-request-mapping-annotations) | [§8 Request Mapping](#8-request-mapping-annotations) |
| Q7 | [@GetMapping vs @RequestMapping?](#8-request-mapping-annotations) | [§8 — composed annotations](#8-request-mapping-annotations) |
| Q8 | [@PathVariable vs @RequestParam?](#9-requestparam-pathvariable-requestbody) | [§9 Path vs Query](#9-requestparam-pathvariable-requestbody) |
| Q9 | [What is @RequestBody?](#9-requestparam-pathvariable-requestbody) | [§9 @RequestBody](#9-requestparam-pathvariable-requestbody) |
| Q10 | [What is @ResponseBody?](#10-responsebody--responseentity) | [§10 @ResponseBody](#10-responsebody--responseentity) |

### Intermediate

| # | Question | Answer in section |
|---|----------|-------------------|
| Q11 | [What is @Autowired?](#14-dependency-injection--autowired) | [§14 @Autowired](#14-dependency-injection--autowired) |
| Q12 | [Constructor vs field injection?](#15-injection-styles--constructor-setter-field) | [§15 Injection Styles](#15-injection-styles--constructor-setter-field) |
| Q13 | [@Qualifier vs @Primary?](#16-qualifier--bean-naming) | [§16 @Qualifier](#16-qualifier--bean-naming) |
| Q14 | [Bean scopes in Spring?](#17-bean-scopes--scope) | [§17 @Scope](#17-bean-scopes--scope) |
| Q15 | [Default bean scope?](#17-bean-scopes--scope) | [§17 — Singleton](#17-bean-scopes--scope) |
| Q16 | [@Component vs @Service vs @Repository?](#18-stereotype-annotations--component-service-repository) | [§18 Stereotypes](#18-stereotype-annotations--component-service-repository) |
| Q17 | [What are meta-annotations?](#5-meta-annotations--composed-annotations) | [§5 Meta-Annotations](#5-meta-annotations--composed-annotations) |
| Q18 | [Is @RestController composed of other annotations?](#5-meta-annotations--composed-annotations) | [§5 — @Controller + @ResponseBody](#5-meta-annotations--composed-annotations) |
| Q19 | [Is @SpringBootApplication composed?](#5-meta-annotations--composed-annotations) | [§5 — breakdown](#5-meta-annotations--composed-annotations) |
| Q20 | [How was Spring configured before annotations?](#4-before-annotations--xml-configuration-era) | [§4 XML Era](#4-before-annotations--xml-configuration-era) |
| Q21 | [What is @InitBinder?](#12-initbinder--propertyeditor) | [§12 @InitBinder](#12-initbinder--propertyeditor) |
| Q22 | [What is PropertyEditor?](#12-initbinder--propertyeditor) | [§12 PropertyEditor](#12-initbinder--propertyeditor) |
| Q23 | [Request param type conversion?](#11-request-parameter-type-conversion) | [§11 Type Conversion](#11-request-parameter-type-conversion) |
| Q24 | [Jackson vs Gson?](#13-json-serialization--jackson-vs-gson) | [§13 Jackson vs Gson](#13-json-serialization--jackson-vs-gson) |
| Q25 | [Which JSON library does Spring Boot use?](#13-json-serialization--jackson-vs-gson) | [§13 — Jackson default](#13-json-serialization--jackson-vs-gson) |

### Advanced

| # | Question | Answer in section |
|---|----------|-------------------|
| Q26 | [What is ResponseEntity?](#10-responsebody--responseentity) | [§10 ResponseEntity](#10-responsebody--responseentity) |
| Q27 | [RetentionPolicy RUNTIME vs SOURCE?](#1-java-annotations-fundamentals) | [§1 Retention](#1-java-annotations-fundamentals) |
| Q28 | [PropertyEditor vs Converter?](#12-initbinder--propertyeditor) | [§12 — modern Converter](#12-initbinder--propertyeditor) |
| Q29 | [@ModelAttribute purpose?](#12-initbinder--propertyeditor) | [§12 @ModelAttribute](#12-initbinder--propertyeditor) |
| Q30 | [How does @RequestBody deserialization work?](#13-json-serialization--jackson-vs-gson) | [§13 Jackson + §9](#13-json-serialization--jackson-vs-gson) |

### Annotation quick map (from video slides)

| Annotation | Composed? | See section |
|------------|-----------|-------------|
| `@SpringBootApplication` | Yes | [§6](#6-springbootapplication) |
| `@Controller` | Yes (`@Component`) | [§7](#7-web-layer--controller--restcontroller) |
| `@RestController` | Yes | [§7](#7-web-layer--controller--restcontroller) |
| `@RequestMapping` | No | [§8](#8-request-mapping-annotations) |
| `@GetMapping` / `@PostMapping` / etc. | Yes | [§8](#8-request-mapping-annotations) |
| `@ResponseBody` | No | [§10](#10-responsebody--responseentity) |
| `@RequestParam` | No | [§9](#9-requestparam-pathvariable-requestbody) |
| `@RequestBody` | No | [§9](#9-requestparam-pathvariable-requestbody) |
| `@PathVariable` | No | [§9](#9-requestparam-pathvariable-requestbody) |
| `@Autowired` | No | [§14](#14-dependency-injection--autowired) |
| `@Qualifier` | No | [§16](#16-qualifier--bean-naming) |
| `@Scope` | No | [§17](#17-bean-scopes--scope) |
| `@InitBinder` | No | [§12](#12-initbinder--propertyeditor) |

### Quick revision checklist

- [ ] Can explain Reflection + how Spring scans annotations → [§2](#2-reflection-in-java), [§3](#3-how-spring-uses-annotations--reflection)
- [ ] Know composed annotations table → [§5](#5-meta-annotations--composed-annotations)
- [ ] Know all request binding annotations → [§9](#9-requestparam-pathvariable-requestbody)
- [ ] Know DI styles and @Qualifier → [§14](#14-dependency-injection--autowired)–[§16](#16-qualifier--bean-naming)
- [ ] Know bean scopes → [§17](#17-bean-scopes--scope)
- [ ] Know Jackson default + @InitBinder → [§12](#12-initbinder--propertyeditor), [§13](#13-json-serialization--jackson-vs-gson)

---

*End of document*
