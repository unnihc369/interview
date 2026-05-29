# Day 2 - Dependency Injection & Spring Boot

---

# What is Dependency?

If one object needs another object:

```text
Dependency exists
```

Example:

```java
Car needs Engine
```

---

# Tight Coupling

```java
class Car {

    Engine e = new Engine();
}
```

Problem:

- Hard to test
- Hard to change

---

# Loose Coupling

Dependency injected externally.

---

# Dependency Injection (DI)

## Definition

Providing required objects from outside instead of creating internally.

---

# Benefits

- Loose coupling
- Better testing
- Easier maintenance
- Easier scalability

---

# Types of Dependency Injection

| Type                  | Description                |
| --------------------- | -------------------------- |
| Constructor Injection | Inject through constructor |
| Setter Injection      | Inject through setter      |
| Field Injection       | Inject directly            |

---

# Constructor Injection (Recommended)

```java
@Component
class Engine {
}

@Component
class Car {

    private final Engine engine;

    @Autowired
    public Car(Engine engine) {
        this.engine = engine;
    }
}
```

---

# Why Constructor Injection Best?

- Immutable dependencies
- Easier unit testing
- Mandatory dependencies ensured

---

# Field Injection

```java
@Autowired
private Engine engine;
```

---

# Problems with Field Injection

- Hard to test
- Reflection used
- Not immutable

---

# Setter Injection

```java
@Autowired
public void setEngine(Engine engine) {
    this.engine = engine;
}
```

Used for optional dependencies.

---

# What is @Autowired?

Automatically injects dependency from Spring IOC container.

---

# IOC Container

IOC = Inversion of Control

Spring manages object creation instead of programmer.

---

# Bean

Object managed by Spring container.

---

# How Spring Creates Beans

Using:

- @Component
- @Service
- @Repository
- @Controller
- @Bean

---

# Stereotype Annotations

| Annotation  | Purpose        |
| ----------- | -------------- |
| @Component  | Generic bean   |
| @Service    | Business logic |
| @Repository | Database layer |
| @Controller | Web layer      |

---

# Bean Lifecycle

```text
Bean Created
↓
Dependency Injected
↓
Init Method
↓
Ready to Use
↓
Destroy Method
```

---

# Real Interview Example

```java
@Service
class PaymentService {

    private final PaymentGateway gateway;

    public PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

---

# Dependency Injection Flow

```text
Application Starts
↓
Spring Scans Classes
↓
Creates Beans
↓
Stores in IOC Container
↓
Injects Dependencies
↓
Application Ready
```

---

# Bean Scopes (Interview Critical)

| Scope | Lifecycle | Use case |
|-------|-----------|----------|
| **singleton** (default) | One instance per Spring container | Services, repositories |
| **prototype** | New instance every injection/getBean | Stateful objects |
| **request** | One per HTTP request (web apps) | Request-scoped DTOs |
| **session** | One per HTTP session | Shopping cart |

```java
@Service
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class ReportGenerator { }
```

**Singleton + Prototype trap:** Injecting prototype into singleton gives one prototype instance — use `ObjectProvider<ReportGenerator>` for fresh instance per call.

---

# @Qualifier — Multiple Implementations

```java
@Service
public class PaymentService {
    public PaymentService(@Qualifier("stripeGateway") PaymentGateway gateway) { }
}
```

Without `@Qualifier` when multiple beans exist → `NoUniqueBeanDefinitionException`.

---

# @Repository vs @Service vs @Controller

| Annotation | Purpose |
|------------|---------|
| `@Repository` | DB layer — **translates SQLException → DataAccessException** |
| `@Service` | Business logic (semantic @Component) |
| `@RestController` | REST API — `@Controller` + `@ResponseBody` |

---

# Spring MVC Request Lifecycle

```text
HTTP Request → Filter chain → DispatcherServlet
  → HandlerMapping → Controller method
  → HttpMessageConverter (JSON) → Response
```

**Full details:** [supplements/sde1_spring_mvc_aop.md](supplements/sde1_spring_mvc_aop.md) — AOP, Filter vs Interceptor, CORS, multipart upload.

---

# Frequently Asked Interview Questions

## Why Constructor Injection Preferred?

- Mandatory dependencies
- Better design
- Easier testing

---

## What is IOC?

Control transferred from programmer to framework.

---

## Difference Between Bean and Object

| Object | Bean |
|--------|------|
| Created manually | Managed by Spring |

---

## What Happens When @Autowired Used?

Spring searches matching bean and injects it.

---

## Default bean scope?

**Singleton** — one instance per IoC container.

---

## Difference between @Component and @Repository?

`@Repository` adds **exception translation** for persistence layer — SQLException becomes Spring `DataAccessException`.

---

# One-Line Revision

```text
DI = inject dependencies; prefer constructor injection.
Bean scopes: singleton (default), prototype, request, session.
@Qualifier resolves multiple beans of same type.
MVC: Filter → DispatcherServlet → Controller → JSON response.
```

---

*End of Day 2 Spring Boot*
