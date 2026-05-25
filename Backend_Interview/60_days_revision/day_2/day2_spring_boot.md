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

| Object           | Bean              |
| ---------------- | ----------------- |
| Created manually | Managed by Spring |

---

## What Happens When @Autowired Used?

Spring searches matching bean and injects it.

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
