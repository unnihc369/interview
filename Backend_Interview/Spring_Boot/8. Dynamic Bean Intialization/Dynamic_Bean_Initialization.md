# @Qualifier, @Primary, @Bean & @Value — Complete Interview Guide

## Table of Contents

1. Introduction
2. Unsatisfied Dependency Problem
3. Why UnsatisfiedDependencyException Occurs
4. Multiple Bean Problem
5. Solution 1 — @Qualifier
6. @Qualifier Deep Dive
7. Solution 2 — @Primary
8. @Primary Deep Dive
9. @Qualifier vs @Primary
10. Dynamic Bean Creation Using @Bean
11. Introduction to @Value
12. @Value Sources
13. Reading Properties from application.properties
14. Dynamic Bean Initialization
15. @Configuration Annotation
16. @Bean Annotation
17. Real Spring Boot Examples
18. Common Startup Logs
19. Frequently Asked Interview Questions
20. Final Mental Model

---

# 1. Introduction

One of the most common Spring Boot interview questions is:

```text
What happens when multiple beans of the same type exist?
```

Example:

```java
Order
   ↑
OnlineOrder

Order
   ↑
OfflineOrder
```

Spring finds:

```text
2 Beans
1 Injection Point
```

and becomes confused.

---

# 2. Unsatisfied Dependency Problem

## Scenario

Interface:

```java
public interface Order {

    void createOrder();
}
```

Implementation 1:

```java
@Component
public class OnlineOrder
        implements Order {

    @Override
    public void createOrder() {

        System.out.println(
            "created Online Order"
        );
    }
}
```

Implementation 2:

```java
@Component
public class OfflineOrder
        implements Order {

    @Override
    public void createOrder() {

        System.out.println(
            "created Offline Order"
        );
    }
}
```

Injection:

```java
@RestController
@RequestMapping("/api")
public class User {

    @Autowired
    Order order;
}
```

---

## Problem

Spring finds:

```text
OnlineOrder
OfflineOrder
```

Both match:

```java
Order
```

Spring cannot decide.

---

## Result

Application Startup Fails

Exception:

```text
NoUniqueBeanDefinitionException
```

or

```text
UnsatisfiedDependencyException
```

---

## Interview Question

What is the primary problem highlighted in this setup?

Answer:

Multiple beans of the same type exist and Spring cannot determine which bean should be injected.

---

# 3. Why UnsatisfiedDependencyException Occurs

Spring Dependency Resolution:

```text
@Autowired Order
        ↓
Find Bean Of Type Order
        ↓
OnlineOrder Found
OfflineOrder Found
        ↓
Ambiguous
        ↓
Exception
```

---

# 4. Multiple Bean Problem

Example:

```java
@Component
class PaytmPayment
        implements Payment {
}
```

```java
@Component
class PhonePePayment
        implements Payment {
}
```

```java
@Autowired
Payment payment;
```

Spring asks:

```text
Which one?
```

No answer.

Application fails.

---

# 5. Solution 1 — @Qualifier

## Purpose

Select exact bean by name.

---

## Implementation

```java
@Component
@Qualifier("onlineOrderObject")
public class OnlineOrder
        implements Order {
}
```

```java
@Component
@Qualifier("offlineOrderObject")
public class OfflineOrder
        implements Order {
}
```

Injection:

```java
@RestController
@RequestMapping("/api")
public class User {

    @Autowired

    @Qualifier(
        "onlineOrderObject"
    )
    Order order;
}
```

---

## Output

```text
created Online Order
```

---

## How It Works

```text
@Autowired Order
        ↓
@Qualifier("onlineOrderObject")
        ↓
Inject OnlineOrder
```

---

# 6. @Qualifier Deep Dive

## Purpose

Used when:

```text
Multiple Beans
Same Type
```

exist.

---

## Constructor Injection Example

```java
@Service
public class UserService {

    private final Order order;

    public UserService(

        @Qualifier(
          "offlineOrderObject"
        )
        Order order
    ) {

        this.order = order;
    }
}
```

---

## Advantages

- Explicit Bean Selection
- Removes Ambiguity
- Easy To Understand

---

## Interview Question

When should @Qualifier be used?

Answer:

Whenever multiple beans of the same type exist and a specific implementation must be injected.

---

# 7. Solution 2 — @Primary

## Purpose

Mark one bean as default.

---

## Example

```java
@Component
@Primary
public class OnlineOrder
        implements Order {
}
```

```java
@Component
public class OfflineOrder
        implements Order {
}
```

Injection:

```java
@Autowired
Order order;
```

Spring automatically chooses:

```text
OnlineOrder
```

---

# 8. @Primary Deep Dive

## Resolution Order

Spring tries:

```text
@Qualifier
      ↓
@Primary
      ↓
Exception
```

---

## Interview Question

Difference between @Primary and @Qualifier?

Answer:

@Primary defines a default bean.

@Qualifier selects a specific bean.

---

# 9. @Qualifier vs @Primary

| Feature            | @Qualifier | @Primary |
| ------------------ | ---------- | -------- |
| Explicit Selection | Yes        | No       |
| Default Bean       | No         | Yes      |
| Multiple Beans     | Yes        | Yes      |
| Highest Priority   | Yes        | No       |

---

# 10. Dynamic Bean Creation Using @Bean

Sometimes bean selection depends on:

- Configuration
- Environment
- Runtime Logic

Example:

```text
Dev Environment
     ↓
OfflineOrder

Production
     ↓
OnlineOrder
```

---

# 11. Introduction to @Value

## Purpose

Inject values into Spring Beans.

---

## Example

```java
@Value("${app.name}")
private String appName;
```

Reads:

```properties
app.name=Spring Boot App
```

---

## Interview Definition

@Value injects values from:

- Property Files
- Environment Variables
- System Variables
- Inline Literals

---

# 12. @Value Sources

## application.properties

```properties
server.port=8080
```

```java
@Value("${server.port}")
private int port;
```

---

## Environment Variable

```java
@Value("${JAVA_HOME}")
private String javaHome;
```

---

## Inline Literal

```java
@Value("Hello Spring")
private String message;
```

---

## Default Value

```java
@Value("${port:8080}")
private int port;
```

If value missing:

```text
8080
```

used.

---

# 13. Reading Properties from application.properties

## application.properties

```properties
isOnlineOrder=false
```

---

## Reading Property

```java
@Value("${isOnlineOrder}")
private boolean isOnlineOrder;
```

---

# 14. Dynamic Bean Initialization

## Requirement

Create:

```text
OnlineOrder
```

or

```text
OfflineOrder
```

based on configuration.

---

## application.properties

```properties
isOnlineOrder=false
```

---

## Configuration Class

```java
@Configuration
public class AppConfig {

    @Bean
    public Order createOrderBean(

        @Value("${isOnlineOrder}")
        boolean isOnlineOrder
    ) {

        if (isOnlineOrder) {

            return new OnlineOrder();

        } else {

            return new OfflineOrder();
        }
    }
}
```

---

## Startup Output

```text
Offline Order Initialized
```

---

## Change Property

```properties
isOnlineOrder=true
```

---

Output:

```text
Online Order Initialized
```

---

# 15. @Configuration Annotation

## Purpose

Marks class as Spring Configuration.

```java
@Configuration
public class AppConfig {
}
```

Equivalent XML:

```xml
<beans>
</beans>
```

---

## Responsibilities

- Bean Creation
- Dependency Configuration
- Dynamic Bean Selection

---

# 16. @Bean Annotation

## Purpose

Creates Bean manually.

---

Example:

```java
@Bean
public Order createOrderBean() {

    return new OnlineOrder();
}
```

Spring registers:

```text
createOrderBean
```

inside IoC Container.

---

## Why Use @Bean?

Used when:

- Third-party Classes
- Dynamic Initialization
- Conditional Logic
- Legacy Classes

---

# 17. Real Spring Boot Example

## application.properties

```properties
isOnlineOrder=false
```

---

## AppConfig

```java
@Configuration
public class AppConfig {

    @Bean
    public Order createOrderBean(

            @Value(
              "${isOnlineOrder}"
            )
            boolean isOnlineOrder
    ) {

        if (isOnlineOrder) {

            return new OnlineOrder();

        }

        return new OfflineOrder();
    }
}
```

---

## Controller

```java
@RestController
@RequestMapping("/api")
public class User {

    @Autowired
    private Order order;

    @PostMapping("/createOrder")
    public ResponseEntity<String>
    createOrder() {

        order.createOrder();

        return ResponseEntity.ok("");
    }
}
```

---

## Output

Property:

```properties
isOnlineOrder=false
```

Output:

```text
Offline Order Initialized
created Offline Order
```

---

Property:

```properties
isOnlineOrder=true
```

Output:

```text
Online Order Initialized
created Online Order
```

---

# 18. Common Startup Logs

## Offline Order

```text
Offline Order Initialized
```

---

Request:

```http
POST
/api/createOrder
```

Output:

```text
created Offline Order
```

---

## Online Order

```text
Online Order Initialized
```

Request:

```http
POST
/api/createOrder
```

Output:

```text
created Online Order
```

---

# 19. Frequently Asked Interview Questions

## Dependency Resolution

1. What is UnsatisfiedDependencyException?
2. Why does NoUniqueBeanDefinitionException occur?
3. What happens if multiple beans of same type exist?
4. How does Spring resolve dependencies?

---

## @Qualifier

5. What is @Qualifier?
6. When should @Qualifier be used?
7. Can @Qualifier be used with Constructor Injection?

---

## @Primary

8. What is @Primary?
9. Difference between @Primary and @Qualifier?
10. Which has higher priority?

---

## @Value

11. What is @Value?
12. What are sources supported by @Value?
13. How to read application.properties values?
14. How to define default values?

---

## @Bean

15. What is @Bean?
16. Why use @Bean instead of @Component?
17. What is @Configuration?

---

## Dynamic Initialization

18. How can Spring create beans dynamically?
19. How can bean selection depend on configuration?
20. How would you switch implementations without changing code?

---

# 20. Final Mental Model

Application Starts

↓

@Configuration Loaded

↓

Read application.properties

↓

@Value Injected

↓

@Bean Method Executed

↓

OnlineOrder OR OfflineOrder Created

↓

Bean Registered In IoC Container

↓

@Autowired Resolves Dependency

↓

Application Ready

---

## One-Line Interview Summary

When multiple beans of the same type exist, Spring throws UnsatisfiedDependencyException due to ambiguity. This can be resolved using @Qualifier for explicit selection, @Primary for default selection, or dynamic bean creation using @Configuration, @Bean, and @Value for runtime flexibility.
