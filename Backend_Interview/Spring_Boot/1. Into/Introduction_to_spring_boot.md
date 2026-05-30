# Introduction to Spring Boot

**Audience:** SDE 1–2 backend interviews · Java developers learning Spring Boot  
**Goal:** Understand *why* Servlet → Spring MVC → Spring Boot evolved, then speak confidently about IoC, DI, request flow, and auto-configuration.

---

## How to Read This Guide

| If you are… | Start here | Time |
|-------------|------------|------|
| **New to Spring** | §1 → §5 → §6 → §7 (evolution + DI + request flow) | ~2–3 hrs |
| **Interview in 1 week** | §8 comparison + §6 DI + §9 cheat sheet + §17 Interview Q&A | ~1 hr |
| **Already know MVC** | §5 Spring Boot + §11 layered architecture + §13 limitations | ~45 min |

**Mental model (one line):**  
Servlet (raw HTTP) → Spring MVC (annotations + DI) → Spring Boot (MVC + starters + auto-config + embedded Tomcat).

---

## Table of Contents

### Part I — Evolution (why Spring Boot exists)

1. [Servlet & Servlet Container](#1-servlet--servlet-container)
2. [Problems with Servlets](#2-problems-with-servlets)
3. [Spring MVC — Solving Servlet Problems](#3-spring-mvc--solving-servlet-problems)
4. [Problems with Spring MVC](#4-problems-with-spring-mvc)
5. [Spring Boot — Solving Spring MVC Problems](#5-spring-boot--solving-spring-mvc-problems)

### Part II — Core concepts (interview-critical)

6. [IoC, Bean, @Autowired & Dependency Injection](#6-ioc-bean-autowired--dependency-injection--full-picture)
7. [Full Request Flow — Spring Boot](#7-full-request-flow--spring-boot)
8. [Quick Comparison Table](#8-quick-comparison-table)
9. [Key Terms — Interview Cheat Sheet](#9-key-terms--interview-cheat-sheet)

### Part III — Framework depth

10. [Spring Framework & Modules](#10-spring-framework--modules)
11. [Layered Architecture (Controller → Service → Repository)](#11-layered-architecture-controller--service--repository)
12. [Spring vs Spring Boot · Spring MVC vs Spring Boot](#12-spring-vs-spring-boot--spring-mvc-vs-spring-boot)
13. [Spring Boot Features, Advantages & Limitations](#13-spring-boot-features-advantages--limitations)
14. [Why Spring Boot for Microservices](#14-why-spring-boot-for-microservices)
15. [Development Flow & Project Layout](#15-development-flow--project-layout)

### Part IV — Practice & interviews

16. [Learning Roadmap](#16-learning-roadmap)
17. [Interview Questions & Answers](#17-interview-questions--answers)
18. [Summary](#18-summary)
19. [One-Page Revision Card](#one-page-revision-card)

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

| Problem                          | Explanation                                                                                                       |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Huge web.xml**                 | Every servlet + every URL mapping goes here. In production apps with 100+ servlets, this becomes unmanageable     |
| **One HTTP method per class**    | Each servlet has only ONE `doGet`, ONE `doPost` — all endpoints share one method, leading to messy if-else chains |
| **Tight coupling**               | Objects are created with `new` inside classes — makes unit testing nearly impossible                              |
| **Hard to unit test**            | You can't mock dependencies that are hardcoded with `new`                                                         |
| **Manual dependency management** | Every library version must be manually tracked and kept compatible                                                |
| **External deployment**          | App must be packaged as a `.war` file and deployed to an external Tomcat server                                   |

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

| Problem                          | Explanation                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------ |
| **Manual dependency versions**   | Developer must pick compatible versions for every library in pom.xml           |
| **Version compatibility issues** | Upgrading one library may break another (e.g., junit 4 → 5 breaks spring-test) |
| **Boilerplate configuration**    | Must create `AppConfig`, `DispatcherServlet` class for every project           |
| **External server required**     | Still must create a `.war` and deploy to external Tomcat                       |
| **Verbose setup**                | Even a simple Hello World needs 3+ configuration files                         |

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

| Feature                      | Servlet                 | Spring MVC                          | Spring Boot                   |
| ---------------------------- | ----------------------- | ----------------------------------- | ----------------------------- |
| Configuration                | web.xml                 | AppConfig + DispatcherServlet class | `@SpringBootApplication` only |
| URL Mapping                  | web.xml tags            | `@RequestMapping` annotations       | `@RequestMapping` annotations |
| Multiple endpoints per class | No (if/else workaround) | Yes (`@GetMapping`, etc.)           | Yes                           |
| Dependency Management        | Manual                  | Manual with versions                | Starter parent handles it     |
| Object Creation              | `new` keyword           | IoC Container                       | IoC Container                 |
| Dependency Injection         | No                      | Yes (`@Autowired`)                  | Yes (`@Autowired`)            |
| Server                       | External Tomcat         | External Tomcat                     | Embedded Tomcat               |
| Deployment                   | `.war` to Tomcat        | `.war` to Tomcat                    | Run as `.jar`                 |
| Unit Testing                 | Very hard               | Easy (mockable)                     | Easy (mockable)               |
| Boilerplate                  | High                    | Medium                              | Minimal                       |

---

## 9. Key Terms — Interview Cheat Sheet

| Term                          | One-liner                                                                                              |
| ----------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Servlet**                   | Java class that handles HTTP request/response                                                          |
| **Servlet Container**         | Runtime (Tomcat) that manages servlets                                                                 |
| **web.xml**                   | Old XML file for URL-to-servlet mapping                                                                |
| **Spring MVC**                | Spring framework for building web apps with annotations                                                |
| **DispatcherServlet**         | Front controller — single entry point for all HTTP requests                                            |
| **Spring Boot**               | Opinionated wrapper over Spring MVC — adds auto-config, starters, embedded server                      |
| **Bean**                      | Any object created and managed by the Spring IoC Container                                             |
| **IoC**                       | Inversion of Control — Spring takes over object creation from you                                      |
| **IoC Container**             | The Spring runtime that stores and manages all beans                                                   |
| **DI (Dependency Injection)** | How Spring provides dependencies to a bean (via constructor/field/setter)                              |
| **@Autowired**                | Tells Spring to inject a matching bean from the IoC container                                          |
| **@Component**                | Marks a class as a Spring-managed bean                                                                 |
| **@Service**                  | `@Component` specialized for business logic layer                                                      |
| **@Repository**               | `@Component` specialized for data access layer                                                         |
| **@Controller**               | `@Component` specialized for web layer (returns views)                                                 |
| **@RestController**           | `@Controller` + automatically serializes return value to JSON                                          |
| **@Qualifier**                | Specifies which bean to inject when multiple match                                                     |
| **@Primary**                  | Marks a bean as the default when multiple candidates exist                                             |
| **Starter Dependency**        | Spring Boot's bundled dependency (e.g., `spring-boot-starter-web`) that auto-includes all related libs |
| **Auto Configuration**        | Spring Boot auto-sets up beans based on classpath — opinionated defaults                               |
| **Embedded Server**           | Tomcat bundled inside the Spring Boot JAR — no external deployment needed                              |
| **Tight Coupling**            | Classes creating dependencies with `new` — hard to test, hard to swap                                  |
| **Loose Coupling**            | Dependencies are injected, not created — easy to test and swap                                         |
| **Bean Scope**                | How many instances Spring creates: singleton (default), prototype, request, session                    |

---

## 10. Spring Framework & Modules

**Spring Framework** is a lightweight container for Java enterprise apps. Goals: **loose coupling**, **dependency injection**, easier testing, maintainability.

Spring Boot **does not replace** Spring — it is an opinionated layer on top.

| Module | Purpose |
|--------|---------|
| **Spring Core** | IoC container, DI, bean lifecycle |
| **Spring MVC** | Web layer — `@Controller`, `DispatcherServlet`, REST |
| **Spring Data JPA** | Database access — repositories, Hibernate integration |
| **Spring Security** | Authentication & authorization |
| **Spring Test** | `@SpringBootTest`, MockMvc, test slices |

```text
Spring Boot app typically uses:
  spring-boot-starter-web      → MVC + embedded Tomcat + Jackson
  spring-boot-starter-data-jpa → Spring Data + Hibernate
  spring-boot-starter-security → Spring Security (optional)
```

---

## 11. Layered Architecture (Controller → Service → Repository)

Standard production layout — **always explain this in interviews**.

```text
Client (Browser / Mobile / Postman)
        ↓
@RestController     ← HTTP, validation, status codes (thin layer)
        ↓
@Service            ← Business rules, transactions
        ↓
@Repository         ← DB access (JPA / JDBC)
        ↓
Database
```

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) { // constructor injection
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        return userService.getById(id);
    }
}

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserResponse getById(Long id) {
        return userRepository.findById(id)
            .map(UserResponse::from)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {}
```

| Layer | Responsibility | Should NOT |
|-------|----------------|------------|
| Controller | HTTP mapping, DTO in/out | Contain business logic or SQL |
| Service | Business rules, orchestration | Know about HTTP details |
| Repository | Persistence | Contain business rules |

---

## 12. Spring vs Spring Boot · Spring MVC vs Spring Boot

### Spring Framework vs Spring Boot

| | Spring Framework | Spring Boot |
|---|------------------|---------------|
| **What** | Core IoC + modules (MVC, Data, Security) | Opinionated setup on top of Spring |
| **Configuration** | XML / Java config — more manual | Auto-configuration + `application.properties` |
| **Server** | Deploy `.war` to external Tomcat | Embedded Tomcat/Jetty in executable JAR |
| **Dependencies** | You pick every version | `spring-boot-starter-*` bundles compatible versions |
| **Use when** | Full control, legacy apps | New REST APIs, microservices (default choice) |

### Spring MVC vs Spring Boot

| | Spring MVC | Spring Boot |
|---|------------|-------------|
| **Scope** | Web framework only | Full application bootstrap |
| **DispatcherServlet** | You configure manually | Auto-configured |
| **Typical entry** | `web.xml` or `WebApplicationInitializer` | `@SpringBootApplication` + `main()` |
| **Relationship** | Spring Boot **includes** Spring MVC via `starter-web` |

**Interview one-liner:** Spring MVC is the web layer; Spring Boot is Spring MVC plus starters, auto-config, and embedded server so you ship a runnable JAR.

---

## 13. Spring Boot Features, Advantages & Limitations

### Features (remember 5)

1. **Auto-configuration** — beans created from classpath (e.g. `DataSource` if JDBC driver present)
2. **Starter dependencies** — one dependency pulls a tested stack
3. **Embedded server** — Tomcat (default), Jetty, Undertow
4. **Actuator** — `/actuator/health`, metrics (production readiness)
5. **Externalized config** — `application.properties` / YAML, profiles (`dev`, `prod`)

### Advantages

| Advantage | Detail |
|-----------|--------|
| Faster development | Less XML, less boilerplate |
| Easy deployment | `java -jar app.jar` |
| Better testing | `@SpringBootTest`, MockMvc, `@DataJpaTest` |
| Cloud-ready | Docker, K8s, 12-factor config |
| Strong ecosystem | Spring Data, Security, Cloud |

### Limitations (balanced answer in interviews)

| Limitation | Detail |
|------------|--------|
| Less explicit control | Magic of auto-config — use `--debug` or `spring-boot-actuator` conditions |
| Memory footprint | Embedded server + full context vs thin servlet |
| Startup time | Large apps — tune lazy init, exclude unused auto-config |
| Opinionated defaults | Must learn conventions to override correctly |

---

## 14. Why Spring Boot for Microservices

| Reason | How Spring Boot helps |
|--------|------------------------|
| **Rapid development** | Starters + auto-config per service |
| **Embedded server** | Each service = one JAR, one process |
| **Externalized config** | Config Server / env vars per service |
| **Spring Cloud** | Eureka, Gateway, Config, OpenFeign on same stack |
| **Actuator** | Health checks for K8s liveness/readiness |
| **Docker-friendly** | Fat JAR → container image |

```text
Typical microservice stack:
  Spring Boot service → Docker → Kubernetes
  Discovery: Eureka / Consul
  API Gateway: Spring Cloud Gateway
  Config: Spring Cloud Config
```

---

## 15. Development Flow & Project Layout

```text
1. Create project (Spring Initializr / start.spring.io)
2. Add starters (web, data-jpa, security, test)
3. Define entities + repositories
4. Implement service layer
5. Expose REST controllers
6. Configure application.properties
7. Write tests (MockMvc, @DataJpaTest)
8. Package: mvn package → java -jar target/app.jar
```

**Standard package layout:**

```text
com.company.myapp/
  MyApplication.java          ← @SpringBootApplication
  controller/
  service/
  repository/
  model/                      ← entities
  dto/                        ← request/response objects
  exception/                  ← @ControllerAdvice handlers
  config/                     ← @Configuration beans
```

**Component scan rule:** Spring scans the package of `main()` and **sub-packages only**. Put `Application.java` at the root package.

---

## 16. Learning Roadmap

| Phase | Topics | Outcome |
|-------|--------|---------|
| 1 | Core Java, OOP, collections | Language foundation |
| 2 | Servlets (optional but helps interviews) | Understand HTTP baseline |
| 3 | Spring Core — IoC, DI, beans | Container mental model |
| 4 | Spring MVC — controllers, REST | Build APIs |
| 5 | **Spring Boot** — starters, auto-config, JAR deploy | This document |
| 6 | Spring Data JPA | Database layer |
| 7 | Spring Security + JWT | Auth |
| 8 | Testing — JUnit, Mockito, MockMvc | Quality |
| 9 | Microservices — Eureka, Gateway | Distributed systems |
| 10 | Docker + basic K8s | Deployment |

**Next docs in this repo:** `2. Create_Project/`, `3. Maven/`, `4. Annotations/`, `60_days_revision/` daily Spring files.

---

## 17. Interview Questions & Answers

### Servlets & HTTP

**Q1. What is a Servlet?**  
A Java class that handles HTTP request/response, extending `HttpServlet` with `doGet`, `doPost`, etc. Mapped to URLs via `web.xml` or `@WebServlet`.

**Q2. What is a Servlet Container?**  
Runtime (e.g. Tomcat) that manages servlet lifecycle, thread pool, and dispatches requests to the correct servlet.

**Q3. Servlet lifecycle?**  
Load → instantiate → `init()` → `service()` (many times) → `destroy()`.

**Q4. GenericServlet vs HttpServlet?**  
`HttpServlet` extends `GenericServlet` and adds HTTP-specific methods (`doGet`, `doPost`).

**Q5. What is `web.xml`?**  
Deployment descriptor mapping URLs to servlet classes (legacy; Spring uses annotations).

---

### Spring Core — IoC & DI

**Q6. What is IoC?**  
Inversion of Control — framework creates and wires objects instead of you using `new`.

**Q7. What is Dependency Injection?**  
Technique to provide dependencies from outside (constructor, setter, field). IoC is the principle; DI is the mechanism.

**Q8. What is a Bean?**  
Object created and managed by the Spring IoC container.

**Q9. Bean lifecycle (high level)?**  
Instantiate → inject dependencies → `@PostConstruct` → use → `@PreDestroy` → destroy.

**Q10. BeanFactory vs ApplicationContext?**  
`ApplicationContext` is superset — adds events, i18n, AOP; use ApplicationContext in Spring Boot.

**Q11. `@Component` vs `@Bean`?**  
`@Component` on a class — auto-detected by scan. `@Bean` on a **method** inside `@Configuration` — manual bean definition.

**Q12. Constructor vs field injection?**  
Prefer **constructor** — immutable deps, required dependencies explicit, easier unit tests without Spring context.

---

### Spring MVC

**Q13. What is DispatcherServlet?**  
Front Controller — single servlet receiving all requests, delegating to controllers via handler mapping.

**Q14. Explain Spring MVC request flow.**  
Client → DispatcherServlet → HandlerMapping → Controller → Service → Repository → response via HttpMessageConverter (JSON).

**Q15. `@Controller` vs `@RestController`?**  
`@RestController` = `@Controller` + `@ResponseBody` on every method — return value written directly to HTTP body (JSON).

**Q16. `@RequestParam` vs `@PathVariable`?**  
`@PathVariable` — part of URL path (`/users/{id}`). `@RequestParam` — query string (`?page=1`).

---

### Spring Boot

**Q17. What is Spring Boot?**  
Opinionated layer on Spring that adds auto-configuration, starter dependencies, and embedded server for fast REST development.

**Q18. Why use Spring Boot over plain Spring MVC?**  
Less boilerplate, managed dependency versions, runnable JAR, production features (Actuator).

**Q19. What is auto-configuration?**  
`@EnableAutoConfiguration` loads config classes from `META-INF/spring/...AutoConfiguration.imports` when classpath conditions match (e.g. JDBC driver → `DataSource`).

**Q20. What are Starters?**  
Dependency descriptors bundling related libraries (e.g. `spring-boot-starter-web` → MVC + Tomcat + Jackson).

**Q21. What is embedded Tomcat?**  
Tomcat runs inside your JAR — no external app server install; `main()` starts the server.

**Q22. What does `@SpringBootApplication` contain?**  
`@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` (default: package of main class).

**Q23. How does Spring Boot start?**  
`SpringApplication.run()` → create ApplicationContext → auto-config → start embedded Tomcat → ready on port 8080.

**Q24. How deploy Spring Boot?**  
`mvn package` → `java -jar app.jar`, or Docker image, or cloud PaaS. No WAR to external Tomcat required.

**Q25. What are Spring profiles?**  
`spring.profiles.active=dev` loads `application-dev.properties` — different DB, logging per environment.

**Q26. What is Actuator?**  
Production endpoints: `/actuator/health`, `/actuator/metrics` — monitoring and K8s probes.

---

## 18. Summary

```text
Traditional Java web     →  Servlets + web.xml + external Tomcat
Spring Framework         →  IoC + DI + Spring MVC (annotations)
Spring Boot              →  MVC + starters + auto-config + embedded server + Actuator
Microservices (typical)  →  Many Spring Boot services + Spring Cloud + Docker/K8s
```

**Interview story (30 seconds):**  
"We moved from servlets with huge web.xml to Spring MVC with DispatcherServlet and dependency injection. Spring Boot keeps MVC but removes version hell and server setup using starters and auto-configuration, so we run `java -jar` with embedded Tomcat and focus on business logic in Controller-Service-Repository layers."

---

## One-Page Revision Card

| Topic | Remember |
|-------|----------|
| Servlet | HTTP handler class; Tomcat = container |
| Spring MVC | Front controller + `@GetMapping` + DI |
| Spring Boot | MVC + starters + auto-config + embedded Tomcat |
| IoC / DI | Spring creates & injects beans |
| `@Autowired` | Inject bean from container (prefer constructor) |
| Request flow | Tomcat → DispatcherServlet → Controller → Service → Repo |
| `@SpringBootApplication` | Config + auto-config + component scan |
| Layering | Controller thin, logic in Service, DB in Repository |

---

*End of Introduction to Spring Boot — aligned with Backend Interview 60-day revision track.*
