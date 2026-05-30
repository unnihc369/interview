# Dependency Injection (DI) & Bean Lifecycle — Complete Interview Guide

## Table of Contents

1. Introduction to Dependency Injection
2. Why Dependency Injection?
3. Tight Coupling vs Loose Coupling
4. Dependency Inversion Principle (DIP)
5. Inversion of Control (IoC)
6. Dependency Injection (DI)
7. Types of Dependency Injection
8. Field Injection
9. Setter Injection
10. Constructor Injection
11. Constructor Injection vs Setter Injection vs Field Injection
12. Bean Lifecycle
13. Bean Lifecycle Callbacks
14. Circular Dependency
15. Solving Circular Dependency
16. Unsatisfied Dependency Problem
17. @Primary Annotation
18. @Qualifier Annotation
19. @Lazy Annotation
20. Spring IoC Container
21. ApplicationContext vs BeanFactory
22. Real Spring Boot Examples
23. Common Interview Questions
24. Final Mental Model

---

# 1. Introduction to Dependency Injection

## What is Dependency Injection?

Dependency Injection (DI) is a design pattern in which the dependencies required by a class are provided from an external source instead of being created inside the class.

Without DI:

```java
public class OrderService {

    private PaymentService paymentService =
            new PaymentService();
}
```

The class creates its own dependency.

---

With DI:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

Spring provides the dependency.

---

## Interview Definition

Dependency Injection is a technique where the Spring IoC Container provides the required dependencies to a class instead of the class creating them itself.

---

# 2. Why Dependency Injection?

Without DI:

```java
OrderService
      ↓
PaymentService
```

OrderService is tightly coupled to PaymentService.

Problems:

- Difficult to test
- Difficult to replace implementations
- Difficult to maintain
- Hard to extend

---

With DI:

```java
OrderService
      ↓
PaymentProcessor
      ↓
PaymentService
```

OrderService depends on an abstraction.

Benefits:

- Loose Coupling
- Easy Testing
- Better Maintainability
- Better Extensibility

---

# 3. Tight Coupling vs Loose Coupling

## Tight Coupling

```java
public class NotificationService {

    EmailService emailService =
            new EmailService();
}
```

Problems:

- Dependency fixed
- Hard to mock
- Hard to replace

---

## Loose Coupling

```java
public interface MessageService {

    void send();
}
```

```java
@Service
public class EmailService
        implements MessageService {

    public void send() {
    }
}
```

```java
@Service
public class NotificationService {

    private final MessageService service;

    public NotificationService(
            MessageService service
    ) {
        this.service = service;
    }
}
```

---

# 4. Dependency Inversion Principle (DIP)

## Definition

DIP is the "D" in SOLID.

### Rule 1

High-level modules should not depend on low-level modules.

Both should depend on abstractions.

---

### Rule 2

Abstractions should not depend on details.

Details should depend on abstractions.

---

Bad:

```java
OrderService
      ↓
PaymentService
```

Good:

```java
OrderService
      ↓
PaymentProcessor
      ↓
PaymentService
```

---

## Interview Question

What principle does Dependency Injection help achieve?

Answer:

Dependency Injection helps achieve the Dependency Inversion Principle (DIP).

---

# 5. Inversion of Control (IoC)

## Traditional Approach

```java
UserService service =
        new UserService();
```

Developer controls object creation.

---

## IoC Approach

```java
@Autowired
UserService service;
```

Spring controls object creation.

---

## Definition

IoC means transferring object creation and lifecycle management from application code to the Spring Container.

---

# 6. Dependency Injection (DI)

DI is the implementation of IoC.

Spring creates:

```java
UserService
```

and injects it where needed.

---

# 7. Types of Dependency Injection

Spring supports:

1. Field Injection
2. Setter Injection
3. Constructor Injection

---

# 8. Field Injection

## Example

```java
@Service
public class UserService {

    @Autowired
    private UserRepository repository;
}
```

---

## Advantages

- Less code
- Easy to write

---

## Disadvantages

### Cannot Use Final Fields

```java
private final UserRepository repository;
```

Not possible.

---

### Difficult Unit Testing

```java
@Mock
UserRepository repository;
```

Harder to inject.

---

### Hidden Dependencies

Not obvious from constructor.

---

### Possible NullPointerException

Can lead to runtime failures.

---

## Interview Verdict

Avoid Field Injection.

---

# 9. Setter Injection

## Example

```java
@Service
public class UserService {

    private UserRepository repository;

    @Autowired
    public void setRepository(
            UserRepository repository
    ) {
        this.repository = repository;
    }
}
```

---

## Advantages

- Dependencies can be modified
- Easier mocking
- Optional dependencies

---

## Disadvantages

- No immutability
- Dependency can change later
- Poor readability in large classes

---

# 10. Constructor Injection

## Example

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(
            UserRepository repository
    ) {
        this.repository = repository;
    }
}
```

---

## Advantages

### Mandatory Dependency

Object cannot exist without dependency.

---

### Supports Final Fields

```java
private final UserRepository repository;
```

---

### Immutable Objects

Dependencies never change.

---

### Easier Unit Testing

```java
UserService service =
        new UserService(mockRepo);
```

---

### Fail Fast

Compilation fails if dependency missing.

---

## Disadvantage

If multiple constructors exist:

```java
@Autowired
public UserService(...) {
}
```

must be specified.

---

## Interview Verdict

Constructor Injection is recommended.

---

# 11. Constructor vs Setter vs Field Injection

| Feature             | Constructor | Setter    | Field     |
| ------------------- | ----------- | --------- | --------- |
| Immutable           | Yes         | No        | No        |
| Final Fields        | Yes         | No        | No        |
| Unit Testing        | Easy        | Easy      | Difficult |
| Optional Dependency | No          | Yes       | Yes       |
| Recommended         | Yes         | Sometimes | No        |

---

# 12. Bean Lifecycle

## Complete Lifecycle

Application Start

↓

Configuration Loaded

↓

IoC Container Started

↓

Construct Bean

↓

Inject Dependency

↓

@PostConstruct

↓

Bean Ready

↓

Bean Used

↓

@PreDestroy

↓

Bean Destroyed

---

# 13. Bean Lifecycle Callbacks

## @PostConstruct

Runs after dependency injection.

```java
@Component
public class DatabaseService {

    @PostConstruct
    public void init() {

        System.out.println(
            "Database Connected"
        );
    }
}
```

---

## @PreDestroy

Runs before bean destruction.

```java
@Component
public class DatabaseService {

    @PreDestroy
    public void cleanup() {

        System.out.println(
            "Closing Connection"
        );
    }
}
```

---

# 14. Circular Dependency

## Problem

```java
@Service
class OrderService {

    private final InvoiceService invoice;
}
```

```java
@Service
class InvoiceService {

    private final OrderService order;
}
```

Dependency Chain:

```text
OrderService
      ↓
InvoiceService
      ↓
OrderService
```

Spring cannot create either bean.

Application startup fails.

---

## Error

```text
BeanCurrentlyInCreationException
```

---

# 15. Solving Circular Dependency

## Solution 1

Refactor Design

Move common logic to:

```java
CommonService
```

---

## Solution 2

Use @Lazy

```java
@Service
public class OrderService {

    private final InvoiceService invoice;

    public OrderService(
        @Lazy InvoiceService invoice
    ) {
        this.invoice = invoice;
    }
}
```

Spring creates proxy object.

Actual bean created later.

---

# 16. Unsatisfied Dependency Problem

## Problem

```java
public interface Order {
}
```

```java
@Service
public class OnlineOrder
        implements Order {
}
```

```java
@Service
public class OfflineOrder
        implements Order {
}
```

Spring finds:

```text
2 Beans
```

Cannot decide which one to inject.

---

## Error

```text
NoUniqueBeanDefinitionException
```

---

# 17. @Primary Annotation

## Purpose

Marks default bean.

```java
@Service
@Primary
public class OnlineOrder
        implements Order {
}
```

Now Spring injects:

```java
OnlineOrder
```

by default.

---

# 18. @Qualifier Annotation

## Purpose

Select specific bean.

```java
@Service
public class OrderService {

    private final Order order;

    public OrderService(

        @Qualifier(
            "offlineOrder"
        )
        Order order
    ) {

        this.order = order;
    }
}
```

Specific bean injected.

---

# 19. @Lazy Annotation

## Purpose

Delays bean creation.

---

Normal Bean

```java
@Service
public class UserService {
}
```

Created during startup.

---

Lazy Bean

```java
@Service
@Lazy
public class UserService {
}
```

Created only when first used.

---

## Benefits

- Faster startup
- Solves circular dependency
- Saves memory

---

# 20. Spring IoC Container

Responsible for:

- Creating Beans
- Managing Beans
- Injecting Dependencies
- Destroying Beans

---

## Types

### BeanFactory

Basic Container

---

### ApplicationContext

Advanced Container

Provides:

- AOP
- Events
- Internationalization
- BeanFactory Features

---

# 21. ApplicationContext vs BeanFactory

| Feature              | BeanFactory | ApplicationContext |
| -------------------- | ----------- | ------------------ |
| DI                   | Yes         | Yes                |
| Bean Lifecycle       | Yes         | Yes                |
| Events               | No          | Yes                |
| AOP                  | No          | Yes                |
| Internationalization | No          | Yes                |
| Recommended          | No          | Yes                |

---

# 22. Real Spring Boot Example

```java
@Repository
public class UserRepository {
}
```

```java
@Service
public class UserService {

    private final UserRepository repository;

    public UserService(
            UserRepository repository
    ) {
        this.repository = repository;
    }
}
```

```java
@RestController
public class UserController {

    private final UserService service;

    public UserController(
            UserService service
    ) {
        this.service = service;
    }
}
```

Flow:

Controller

↓

Service

↓

Repository

↓

Database

---

# 23. Frequently Asked Interview Questions

## Basics

1. What is Dependency Injection?
2. What is IoC?
3. Difference between IoC and DI?
4. What is Dependency Inversion Principle?
5. Why is DI needed?

---

## Injection Types

6. What is Field Injection?
7. What is Setter Injection?
8. What is Constructor Injection?
9. Which injection type is recommended?
10. Why Constructor Injection is preferred?

---

## Bean Lifecycle

11. Explain Bean Lifecycle.
12. What is @PostConstruct?
13. What is @PreDestroy?
14. When are dependencies injected?

---

## Dependency Problems

15. What is Circular Dependency?
16. How to solve Circular Dependency?
17. What is @Lazy?
18. What is Unsatisfied Dependency?
19. What is NoUniqueBeanDefinitionException?

---

## Bean Selection

20. What is @Primary?
21. What is @Qualifier?
22. Difference between @Primary and @Qualifier?

---

## Spring Container

23. What is IoC Container?
24. BeanFactory vs ApplicationContext?
25. How does Spring create Beans?

---

# 24. Final Mental Model

Application Start

↓

Spring IoC Container

↓

Scan Components

↓

Create Beans

↓

Inject Dependencies

↓

@PostConstruct

↓

Application Running

↓

Bean Usage

↓

@PreDestroy

↓

Application Shutdown

---

## One-Line Interview Summary

Dependency Injection is a Spring mechanism that implements the Dependency Inversion Principle by allowing the IoC Container to create and inject dependencies, resulting in loosely coupled, testable, and maintainable applications.
