# Introduction to Spring Boot

## Table of Contents
1. [Servlet & Servlet Container](#1-servlet--servlet-container)
2. [Problems with Servlets](#2-problems-with-servlets)
3. [Spring MVC — Solving Servlet Problems](#3-spring-mvc--solving-servlet-problems)
4. [Problems with Spring MVC](#4-problems-with-spring-mvc)
5. [Spring Boot — Solving Spring MVC Problems](#5-spring-boot--solving-spring-mvc-problems)
6. [IoC, Bean, @Autowired & Dependency Injection — Full Picture](#6-ioc-bean-autowired--dependency-injection--full-picture)
7. [Full Request Flow — Spring Boot](#7-full-request-flow--spring-boot)
8. [Quick Comparison Table](#8-quick-comparison-table)

---

## 1. Servlet & Servlet Container

### What is a Servlet?
A **Servlet** is a Java class that:
- Receives an HTTP request
- Processes the logic
- Returns an HTTP response

Think of it as your API handler class in plain Java.

```java
// web.xml mapping (old way)
// URL: /demo-servlet-1  →  handled by DemoServlet1

@WebServlet("/demo-servlet-1")
public class DemoServlet1 extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        String path = request.getPathInfo();

        if ("/first-endpoint".equals(path)) {
            response.getWriter().write("Response from first endpoint");
        } else if ("/second-endpoint".equals(path)) {
            response.getWriter().write("Response from second endpoint");
        }
        // ... more if-else for every endpoint
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        // handle POST
    }

    @Override
    protected void doPut(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        // handle PUT
    }
}

// Another servlet for different URL group
@WebServlet("/demo-servlet-2")
public class DemoServlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        // ...
    }
}
```

### What is a Servlet Container?
A **Servlet Container** (e.g., **Tomcat**) is the runtime that:
- Manages the lifecycle of all servlets
- Uses `web.xml` to map incoming URLs to the correct servlet
- Invokes the right method (`doGet`, `doPost`, etc.) based on the HTTP method

### How web.xml worked
```xml
<!-- web.xml — the old configuration file -->
<web-app>

    <!-- Declare servlet -->
    <servlet>
        <servlet-name>demoServlet1</servlet-name>
        <servlet-class>com.example.DemoServlet1</servlet-class>
    </servlet>

    <!-- Map URL to servlet -->
    <servlet-mapping>
        <servlet-name>demoServlet1</servlet-name>
        <url-pattern>/demo-servlet-1/*</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>demoServlet2</servlet-name>
        <servlet-class>com.example.DemoServlet2</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>demoServlet2</servlet-name>
        <url-pattern>/demo-servlet-2/*</url-pattern>
    </servlet-mapping>

</web-app>
```

### Request Flow (Servlet era)
```
Client Request
     ↓
  Tomcat (Servlet Container)
     ↓  reads web.xml
  Finds correct Servlet
     ↓
  Invokes doGet() / doPost() / doPut() / doDelete()
     ↓
  Servlet processes & sends response
```

---

## 2. Problems with Servlets

| Problem | Explanation |
|---|---|
| **Huge web.xml** | Every servlet + every URL mapping goes here. In production apps with 100+ servlets, this becomes unmanageable |
| **One HTTP method per class** | Each servlet has only ONE `doGet`, ONE `doPost` — all endpoints share one method, leading to messy if-else chains |
| **Tight coupling** | Objects are created with `new` inside classes — makes unit testing nearly impossible |
| **Hard to unit test** | You can't mock dependencies that are hardcoded with `new` |
| **Manual dependency management** | Every library version must be manually tracked and kept compatible |
| **External deployment** | App must be packaged as a `.war` file and deployed to an external Tomcat server |

---

## 3. Spring MVC — Solving Servlet Problems

Spring MVC is part of the Spring Framework family and was introduced to fix the problems above.

### Problem 1 Solved — Remove web.xml (Annotation-based config)

```java
@Controller
@RequestMapping("/payment")
public class PaymentController {

    // ONE method per endpoint — clean and organized
    @GetMapping("/first-endpoint")
    public String getFirstEndpoint() {
        return "Response from first endpoint";
    }

    @GetMapping("/second-endpoint")
    public String getSecondEndpoint() {
        return "Response from second endpoint";
    }

    @PostMapping("/create")
    public String createPayment() {
        return "Payment created";
    }

    @PutMapping("/update")
    public String updatePayment() {
        return "Payment updated";
    }

    @DeleteMapping("/delete")
    public String deletePayment() {
        return "Payment deleted";
    }
}
```

### Problem 2 Solved — DispatcherServlet (Front Controller)

Spring MVC introduces a **DispatcherServlet**, which acts as the single entry point (also called **Front Controller**).

```
Client Request
     ↓
  Tomcat (Servlet Container)
     ↓
  DispatcherServlet  ← single entry point for ALL requests
     ↓
  Handler Mapping    ← uses @RequestMapping/@GetMapping etc. to find the right controller+method
     ↓
  Creates Controller instance via IoC
     ↓
  Injects dependencies via @Autowired
     ↓
  Invokes the specific method
     ↓
  Returns response
```

### Spring MVC — Required Files

```java
// 1. AppConfig.java — Spring configuration
@Configuration
@EnableWebMvc
@ComponentScan("com.example")   // tell Spring where to scan for components
public class AppConfig {
    // Spring loads all required beans from this package
}
```

```java
// 2. MyDispatcherServlet.java — Register the dispatcher servlet
public class MyDispatcherServlet extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() { return null; }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{ AppConfig.class };   // link to our config
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};   // handle all requests
    }
}
```

```xml
<!-- pom.xml — must manually add dependencies WITH versions -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.20</version>          <!-- must specify version -->
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>           <!-- must keep compatible -->
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>          <!-- must track manually -->
</dependency>
```

---

## 4. Problems with Spring MVC

| Problem | Explanation |
|---|---|
| **Manual dependency versions** | Developer must pick compatible versions for every library in pom.xml |
| **Version compatibility issues** | Upgrading one library may break another (e.g., junit 4 → 5 breaks spring-test) |
| **Boilerplate configuration** | Must create `AppConfig`, `DispatcherServlet` class for every project |
| **External server required** | Still must create a `.war` and deploy to external Tomcat |
| **Verbose setup** | Even a simple Hello World needs 3+ configuration files |

---

## 5. Spring Boot — Solving Spring MVC Problems

Spring Boot = Spring MVC + **3 key additions**:
1. **Dependency Management** (starter dependencies)
2. **Auto Configuration** (opinionated defaults)
3. **Embedded Server** (no more `.war` + external Tomcat)

> Spring Boot does NOT replace Spring MVC. It builds ON TOP of it.
> All features of Spring MVC still exist — Spring Boot just removes the boilerplate.

---

### Advantage 1 — Dependency Management (Starter Dependencies)

```xml
<!-- Spring Boot pom.xml — simple and clean -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>    <!-- ONLY version you define -->
</parent>

<dependencies>
    <!-- This ONE starter internally adds spring-webmvc, jackson,
         tomcat-embed, spring-context, and 15+ other compatible deps -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- NO version needed — parent manages it -->
    </dependency>

    <!-- This ONE starter internally adds junit, mockito,
         spring-test and all testing deps -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**How it works internally:**
```
spring-boot-starter-web
    ├── spring-webmvc
    ├── spring-web
    ├── spring-context
    ├── tomcat-embed-core       ← embedded server
    ├── tomcat-embed-websocket
    ├── jackson-databind        ← JSON conversion
    └── ... (15+ more compatible deps)
```

---

### Advantage 2 — Auto Configuration

Spring Boot uses an **opinionated** approach — it provides sensible defaults so you don't have to configure things yourself.

No more `AppConfig.java`. No more `DispatcherServlet` class.

The single annotation `@SpringBootApplication` replaces all of it:

```java
// @SpringBootApplication internally contains:
// @Configuration         — marks this as config class
// @EnableAutoConfiguration — loads all auto-config from starter deps
// @ComponentScan         — scans from this package downwards (default)
// + sets up DispatcherServlet automatically

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
        // Starts embedded Tomcat, sets up Spring context, scans components
    }
}
```

```java
// That's it! Just write your controller — everything else is auto-configured
@RestController
@RequestMapping("/api")
public class MyController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Spring Boot!";
    }
}
```

**Opinionated means:**
- Default component scan starts from the package of `main()` class
- Default embedded server is Tomcat on port `8080`
- Default JSON conversion uses Jackson

**If you disagree with the default, override it:**
```java
@SpringBootApplication
@ComponentScan("com.custom.package")   // override default component scan
public class MyApplication { ... }
```

```properties
# application.properties — override any default
server.port=9090
server.servlet.context-path=/myapp
```

---

### Advantage 3 — Embedded Server

No more creating a `.war` file. No more deploying to external Tomcat.

```
Old way (Servlet / Spring MVC):
  Code → Build .war file → Deploy to external Tomcat → Run Tomcat

Spring Boot way:
  Code → Run main() → Done ✓
  (Tomcat is embedded inside the JAR itself)
```

```bash
# Spring Boot app runs like any Java program
mvn spring-boot:run

# OR build a fat JAR and run anywhere (no Tomcat needed on server)
mvn package
java -jar target/my-app-1.0.jar
```

**Startup log you'll see:**
```
Tomcat started on port(s): 8080 (http)
Started MyApplication in 2.3 seconds
```

---

### Complete Spring Boot "Hello World"

```java
// 1. Main class — entire Spring configuration in one annotation
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

```java
// 2. Controller — just write the business logic
@RestController
@RequestMapping("/my-api")
public class MyController {

    @GetMapping("/first-api")
    public String firstApi() {
        return "Hello from Concept and Coding!";
    }
}
```

```xml
<!-- 3. pom.xml — minimal -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**Hit in browser:**
```
GET http://localhost:8080/my-api/first-api
→ Hello from Concept and Coding!
```

---

## 6. IoC, Bean, @Autowired & Dependency Injection — Full Picture

This is the **heart of Spring**. Understanding these 4 concepts and how they relate is critical for interviews.

---

### The Problem — Tight Coupling

```java
// BAD — tight coupling
public class PaymentService {

    // Creating object directly using 'new'
    private UserService userService = new UserService();   // HARDCODED

    public String getSenderDetails(int id) {
        return userService.getUserDetail(id);
    }
}
```

**Why is `new` a problem?**

1. **Unit testing is broken** — you CANNOT mock `UserService` because it's hardcoded
2. **Cannot swap implementations** — if you want `MockUserService` in tests, impossible
3. **Objects are tightly coupled** — `PaymentService` is responsible for creating its own dependency

```java
// Unit test attempt — FAILS because of tight coupling
@Test
public void testGetSenderDetails() {
    PaymentService service = new PaymentService();
    // We want to mock UserService but CAN'T — it's hardcoded inside
    // Every test call will actually call UserService.getUserDetail()
}
```

---

### The Solution — Dependency Injection (DI)

Instead of creating dependencies, **inject them from outside**.

```java
// GOOD — loose coupling via constructor injection
public class PaymentService {

    private UserService userService;   // NOT created here

    // Dependency is INJECTED from outside
    public PaymentService(UserService userService) {
        this.userService = userService;
    }

    public String getSenderDetails(int id) {
        return userService.getUserDetail(id);
    }
}

// Now in tests, you can inject a mock:
@Test
public void testGetSenderDetails() {
    UserService mockUserService = mock(UserService.class);
    when(mockUserService.getUserDetail(1)).thenReturn("Alice");

    PaymentService service = new PaymentService(mockUserService); // inject mock
    String result = service.getSenderDetails(1);
    assertEquals("Alice", result);   // works perfectly
}
```

**That's the concept of DI.** Spring automates this for you.

---

### Concept 1 — Bean

A **Bean** is simply any object that is **created and managed by Spring** (not by you with `new`).

```java
// This is a regular Java object — NOT a bean (you manage it)
UserService userService = new UserService();

// This is a Spring Bean — Spring manages it
// Spring creates the object, stores it, and injects it where needed
```

How to make a class a Bean — use **stereotype annotations**:

```java
@Component        // generic Spring-managed bean
public class UserService { ... }

@Service          // business logic layer (semantic alias for @Component)
public class PaymentService { ... }

@Repository       // data access layer (semantic alias for @Component)
public class UserRepository { ... }

@Controller       // web layer — handles HTTP requests
public class UserController { ... }

@RestController   // @Controller + @ResponseBody (returns JSON directly)
public class ApiController { ... }
```

> `@Service`, `@Repository`, `@Controller`, `@RestController` are all
> specializations of `@Component`. They all make the class a Spring Bean.
> The difference is semantic — they describe the role of the class.

You can also declare a Bean manually using `@Bean` in a config class:

```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();   // Spring will manage this object
    }

    @Bean
    public EmailService emailService() {
        return new EmailService("smtp.gmail.com");   // with custom config
    }
}
```

---

### Concept 2 — IoC Container (Inversion of Control)

**IoC** means: instead of YOU creating objects, you give that **control to Spring**.

The **IoC Container** is the core of Spring. It is responsible for:
1. Scanning your code for beans (`@Component`, `@Service`, etc.)
2. Creating instances of those beans
3. Managing their lifecycle (creation → usage → destruction)
4. Resolving and injecting dependencies

The IoC Container is represented by the `ApplicationContext` interface:

```java
// In Spring Boot, the IoC container is created automatically at startup
// But you can access it if needed:

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(Application.class, args);

        // Get a bean from the container manually (rarely needed)
        PaymentService ps = context.getBean(PaymentService.class);
        ps.getSenderDetails(1);
    }
}
```

**Before IoC (you control object creation):**
```java
UserService userService = new UserService();
PaymentService ps = new PaymentService(userService);
EmailService email = new EmailService(userService);
// You manage everything — what if UserService changes?
```

**After IoC (Spring controls object creation):**
```java
// Spring creates UserService, PaymentService, EmailService
// Spring knows PaymentService needs UserService — it injects it
// You just write business logic
```

---

### Concept 3 — @Autowired

`@Autowired` is the annotation that tells Spring:
> "I need this dependency — please inject it for me."

Spring will look in the IoC container for a matching bean and inject it automatically.

**Three ways to use @Autowired:**

```java
@Service
public class PaymentService {

    // ===== Method 1: Field Injection =====
    // Simple but not recommended (hard to test, hides dependencies)
    @Autowired
    private UserService userService;


    // ===== Method 2: Constructor Injection (RECOMMENDED) =====
    // Best practice — dependencies are explicit, easy to test
    // In Spring Boot 3+, @Autowired is optional on constructors
    private final UserService userService;

    @Autowired   // optional if there's only one constructor
    public PaymentService(UserService userService) {
        this.userService = userService;
    }


    // ===== Method 3: Setter Injection =====
    // Use when dependency is optional
    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

**What Spring does when it sees @Autowired:**
```
Spring sees: PaymentService needs UserService

Step 1: Look in IoC container — do I have a UserService bean?
Step 2: Found UserService bean (because it's @Component/@Service)
Step 3: Inject that bean into PaymentService
Step 4: PaymentService is ready to use
```

---

### Concept 4 — How all 4 fit together

```
Your Code:
  @Service                    ← tells Spring: "make this a Bean"
  public class UserService { ... }

  @Service
  public class PaymentService {
      @Autowired              ← tells Spring: "inject UserService here"
      private UserService userService;
  }

Spring Boot Startup:
  1. Scans all packages from main class downwards
  2. Finds @Service on UserService → creates Bean, stores in IoC Container
  3. Finds @Service on PaymentService → needs to create Bean
  4. Sees @Autowired on userService field → looks in IoC Container
  5. Finds UserService bean → injects it into PaymentService
  6. PaymentService bean is now ready in IoC Container

Result:
  IoC Container holds:
    { UserService: <instance>, PaymentService: <instance with UserService injected> }
```

```
┌──────────────────────────────────────────────────────┐
│                   IoC Container                       │
│                                                        │
│   ┌─────────────────┐    ┌──────────────────────────┐ │
│   │   UserService   │    │     PaymentService        │ │
│   │   (Bean)        │───▶│     (Bean)                │ │
│   │                 │    │  @Autowired UserService   │ │
│   └─────────────────┘    └──────────────────────────┘ │
│                                                        │
│   Created by Spring  ←── @Component / @Service        │
│   Injected by Spring ←── @Autowired                   │
└──────────────────────────────────────────────────────┘
```

---

### Full Example — IoC + DI + @Autowired + Beans together

```java
// UserService.java — declare as Bean
@Service
public class UserService {

    public String getUserDetail(int id) {
        // In real app, this would query database
        return "User with ID: " + id;
    }
}
```

```java
// EmailService.java — another Bean
@Service
public class EmailService {

    public void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}
```

```java
// PaymentService.java — Bean with injected dependencies
@Service
public class PaymentService {

    // Spring injects both — no 'new' anywhere
    private final UserService userService;
    private final EmailService emailService;

    @Autowired
    public PaymentService(UserService userService, EmailService emailService) {
        this.userService = userService;
        this.emailService = emailService;
    }

    public String processPayout(int senderId) {
        String senderDetails = userService.getUserDetail(senderId);
        emailService.sendEmail("Payout processed for: " + senderDetails);
        return "Payout done for: " + senderDetails;
    }
}
```

```java
// PaymentController.java — Bean with injected PaymentService
@RestController
@RequestMapping("/payment")
public class PaymentController {

    private final PaymentService paymentService;

    @Autowired
    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @GetMapping("/payout/{id}")
    public String payout(@PathVariable int id) {
        return paymentService.processPayout(id);
    }
}
```

```java
// Application.java — Spring Boot entry point
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        // Spring IoC Container starts
        // Scans and finds: UserService, EmailService, PaymentService, PaymentController
        // Creates all beans, resolves all @Autowired dependencies
        // Starts embedded Tomcat on port 8080
    }
}
```

**Request flow:**
```
GET /payment/payout/42
        ↓
  DispatcherServlet (auto-configured)
        ↓
  PaymentController.payout(42)   ← Spring injected PaymentService
        ↓
  PaymentService.processPayout(42) ← Spring injected UserService + EmailService
        ↓
  UserService.getUserDetail(42)
        ↓
  EmailService.sendEmail(...)
        ↓
  "Payout done for: User with ID: 42"
```

---

### Unit Testing with DI (why it matters)

```java
@ExtendWith(MockitoExtension.class)
class PaymentServiceTest {

    @Mock
    private UserService userService;    // Mockito creates a mock

    @Mock
    private EmailService emailService;  // Mockito creates a mock

    @InjectMocks
    private PaymentService paymentService; // injects mocks into PaymentService

    @Test
    void testProcessPayout() {
        // Arrange — define what mock should return
        when(userService.getUserDetail(1)).thenReturn("Alice");

        // Act
        String result = paymentService.processPayout(1);

        // Assert
        assertEquals("Payout done for: Alice", result);
        verify(emailService).sendEmail("Payout processed for: Alice");
    }
}
// Works perfectly because PaymentService uses DI, not 'new'
// If PaymentService used 'new UserService(), this would be impossible
```

---

### Bean Scope

By default, Spring creates ONE instance of each bean (Singleton). You can change this:

```java
@Service
@Scope("singleton")    // DEFAULT — one instance per Spring container
public class UserService { }

@Service
@Scope("prototype")    // new instance every time it's injected/requested
public class RequestProcessor { }

@Service
@Scope("request")      // one instance per HTTP request (web apps only)
public class UserSession { }

@Service
@Scope("session")      // one instance per HTTP session (web apps only)
public class ShoppingCart { }
```

---

### @Qualifier — When Multiple Beans of Same Type Exist

```java
public interface NotificationService {
    void notify(String message);
}

@Service("emailNotification")
public class EmailNotificationService implements NotificationService {
    public void notify(String message) { /* send email */ }
}

@Service("smsNotification")
public class SmsNotificationService implements NotificationService {
    public void notify(String message) { /* send SMS */ }
}

@Service
public class OrderService {

    private final NotificationService notificationService;

    @Autowired
    public OrderService(@Qualifier("smsNotification") NotificationService notificationService) {
        this.notificationService = notificationService;  // injects SMS service
    }
}
```

---

### @Primary — Default Bean When Multiple Candidates Exist

```java
@Service
@Primary   // this will be injected by default when type is ambiguous
public class EmailNotificationService implements NotificationService { ... }

@Service
public class SmsNotificationService implements NotificationService { ... }

@Service
public class OrderService {
    @Autowired   // EmailNotificationService is injected (it's @Primary)
    private NotificationService notificationService;
}
```

---

## 7. Full Request Flow — Spring Boot

```
Client sends:  GET http://localhost:8080/payment/payout/42

1. Embedded Tomcat receives the request

2. DispatcherServlet (auto-configured by Spring Boot)
   - Acts as Front Controller
   - Receives ALL requests

3. Handler Mapping
   - Checks all @GetMapping, @PostMapping, etc. annotations
   - Finds: PaymentController.payout() matches /payment/payout/{id}

4. IoC Container
   - Checks if PaymentController bean exists (it was created at startup)
   - It does → uses the existing singleton instance

5. Invokes PaymentController.payout(42)
   - All dependencies (PaymentService) already injected via @Autowired

6. PaymentService.processPayout(42)
   - Calls UserService.getUserDetail(42)
   - Calls EmailService.sendEmail(...)

7. Returns String response
   - DispatcherServlet writes it to HTTP response

8. Client receives: "Payout done for: User with ID: 42"
```

---

## 8. Quick Comparison Table

| Feature | Servlet | Spring MVC | Spring Boot |
|---|---|---|---|
| Configuration | web.xml | AppConfig + DispatcherServlet class | `@SpringBootApplication` only |
| URL Mapping | web.xml tags | `@RequestMapping` annotations | `@RequestMapping` annotations |
| Multiple endpoints per class | No (if/else workaround) | Yes (`@GetMapping`, etc.) | Yes |
| Dependency Management | Manual | Manual with versions | Starter parent handles it |
| Object Creation | `new` keyword | IoC Container | IoC Container |
| Dependency Injection | No | Yes (`@Autowired`) | Yes (`@Autowired`) |
| Server | External Tomcat | External Tomcat | Embedded Tomcat |
| Deployment | `.war` to Tomcat | `.war` to Tomcat | Run as `.jar` |
| Unit Testing | Very hard | Easy (mockable) | Easy (mockable) |
| Boilerplate | High | Medium | Minimal |

---

## Key Terms — Interview Cheat Sheet

| Term | One-liner |
|---|---|
| **Servlet** | Java class that handles HTTP request/response |
| **Servlet Container** | Runtime (Tomcat) that manages servlets |
| **web.xml** | Old XML file for URL-to-servlet mapping |
| **Spring MVC** | Spring framework for building web apps with annotations |
| **DispatcherServlet** | Front controller — single entry point for all HTTP requests |
| **Spring Boot** | Opinionated wrapper over Spring MVC — adds auto-config, starters, embedded server |
| **Bean** | Any object created and managed by the Spring IoC Container |
| **IoC** | Inversion of Control — Spring takes over object creation from you |
| **IoC Container** | The Spring runtime that stores and manages all beans |
| **DI (Dependency Injection)** | How Spring provides dependencies to a bean (via constructor/field/setter) |
| **@Autowired** | Tells Spring to inject a matching bean from the IoC container |
| **@Component** | Marks a class as a Spring-managed bean |
| **@Service** | `@Component` specialized for business logic layer |
| **@Repository** | `@Component` specialized for data access layer |
| **@Controller** | `@Component` specialized for web layer (returns views) |
| **@RestController** | `@Controller` + automatically serializes return value to JSON |
| **@Qualifier** | Specifies which bean to inject when multiple match |
| **@Primary** | Marks a bean as the default when multiple candidates exist |
| **Starter Dependency** | Spring Boot's bundled dependency (e.g., `spring-boot-starter-web`) that auto-includes all related libs |
| **Auto Configuration** | Spring Boot auto-sets up beans based on classpath — opinionated defaults |
| **Embedded Server** | Tomcat bundled inside the Spring Boot JAR — no external deployment needed |
| **Tight Coupling** | Classes creating dependencies with `new` — hard to test, hard to swap |
| **Loose Coupling** | Dependencies are injected, not created — easy to test and swap |
| **Bean Scope** | How many instances Spring creates: singleton (default), prototype, request, session |
