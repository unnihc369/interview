# Spring Bean Scopes — Complete Interview Guide

## Table of Contents

1. Introduction to Bean Scopes
2. Why Bean Scopes Matter
3. Spring Bean Lifecycle vs Bean Scope
4. Singleton Scope
5. Prototype Scope
6. Request Scope
7. Session Scope
8. Application Scope
9. WebSocket Scope
10. Singleton vs Prototype
11. Request vs Session Scope
12. Scope Initialization (Eager vs Lazy)
13. Scoped Proxy Problem
14. Scoped Proxy Solution
15. Injecting Short-Lived Beans into Long-Lived Beans
16. Session Management and Logout
17. Real Spring Boot Examples
18. Common Interview Questions
19. Final Mental Model

---

# 1. Introduction to Bean Scopes

## What is Bean Scope?

Bean Scope determines:

```text
How many objects Spring creates
and
How long those objects live
```

Example:

```java
@Component
public class User {
}
```

Spring creates a bean.

Scope determines:

- Single object?
- Multiple objects?
- One per request?
- One per session?

---

# 2. Why Bean Scopes Matter

Different applications need different lifecycles.

Examples:

### Database Service

```java
@Service
public class DatabaseService {
}
```

Only one instance required.

Use:

```java
singleton
```

---

### Shopping Cart

Every user needs separate data.

Use:

```java
session
```

---

### HTTP Request Tracking

Every request needs separate object.

Use:

```java
request
```

---

# 3. Spring Bean Lifecycle vs Bean Scope

Bean Lifecycle:

```text
Create Bean
↓
Inject Dependencies
↓
@PostConstruct
↓
Use Bean
↓
@PreDestroy
↓
Destroy Bean
```

Scope decides:

```text
WHEN bean is created
HOW MANY instances exist
WHEN bean is destroyed
```

---

# 4. Singleton Scope

## Definition

Default Spring Bean Scope.

Only one instance exists per IoC Container.

---

## Annotation

```java
@Component
@Scope("singleton")
public class User {
}
```

---

## Example

```java
@Component
@Scope("singleton")
public class User {

    public User() {

        System.out.println(
            "User initialization"
        );
    }

    @PostConstruct
    public void init() {

        System.out.println(
            "User object hashCode : "
                    + this.hashCode()
        );
    }
}
```

---

## Characteristics

### One Object

```text
Only one instance
```

---

### Shared Across Application

```text
Controller
     ↓
Service
     ↓
Repository
```

All use same object.

---

### Eager Initialization

Created during startup.

```text
Application Start
      ↓
Singleton Beans Created
```

---

## Observation

Output:

```text
User initialization
User object hashCode : 123456
```

Printed only once.

---

## Interview Question

What are the characteristics of Singleton Scope?

Answer:

- Default scope
- One object per IoC Container
- Shared across application
- Eagerly initialized

---

# 5. Prototype Scope

## Definition

Spring creates a new object every time it is requested.

---

## Annotation

```java
@Component
@Scope("prototype")
public class User {
}
```

---

## Example

```java
@Component
@Scope("prototype")
public class User {

    public User() {

        System.out.println(
            "User initialization"
        );
    }

    @PostConstruct
    public void init() {

        System.out.println(
            "User object hashCode : "
                    + this.hashCode()
        );
    }
}
```

---

## Characteristics

### Multiple Objects

Every request:

```java
context.getBean(User.class);
```

creates new object.

---

### Lazy Initialization

Created only when requested.

---

## Example

```java
User user1 =
        context.getBean(User.class);

User user2 =
        context.getBean(User.class);
```

Output:

```text
User object hashCode : 123
User object hashCode : 456
```

Different objects.

---

## Observation

Each bean request generates:

```text
New Object
```

---

## Interview Question

Explain Prototype Scope.

Answer:

Prototype scope creates a new bean instance every time Spring is asked for that bean.

---

# 6. Request Scope

## Definition

One object per HTTP Request.

---

## Annotation

```java
@Component
@Scope("request")
public class User {
}
```

---

## Example

```java
@Component
@Scope("request")
public class User {

    public User() {

        System.out.println(
            "User initialization"
        );
    }

    @PostConstruct
    public void init() {

        System.out.println(
            "User object hashCode : "
                    + this.hashCode()
        );
    }
}
```

---

## Request Flow

Request 1

↓

User Bean A

---

Request 2

↓

User Bean B

---

## Characteristics

- New object per request
- Lazy initialized
- Destroyed after request completion

---

## Use Cases

- Request Tracking
- Request Metadata
- Temporary User Context

---

# 7. Session Scope

## Definition

One object per User Session.

---

## Annotation

```java
@Component
@Scope("session")
public class User {
}
```

---

## Characteristics

### One Bean Per User

User A:

```text
Session Bean A
```

User B:

```text
Session Bean B
```

---

### Lazy Initialization

Created when first endpoint accessed.

---

### Long Lifetime

Lives until:

```text
Session Timeout
OR
Logout
```

---

# 8. Application Scope

## Definition

One object per ServletContext.

---

## Annotation

```java
@Component
@Scope("application")
public class AppConfig {
}
```

---

## Lifetime

Entire application lifetime.

---

# 9. WebSocket Scope

## Definition

One bean per WebSocket session.

---

## Annotation

```java
@Scope("websocket")
```

---

Used in real-time applications.

---

# 10. Singleton vs Prototype

| Feature        | Singleton | Prototype |
| -------------- | --------- | --------- |
| Objects        | One       | Multiple  |
| Initialization | Eager     | Lazy      |
| Memory Usage   | Low       | Higher    |
| Performance    | Better    | Lower     |
| Default Scope  | Yes       | No        |

---

# 11. Request vs Session Scope

| Feature       | Request       | Session       |
| ------------- | ------------- | ------------- |
| Lifetime      | One Request   | One Session   |
| Creation      | Every Request | First Request |
| Destruction   | End Request   | Session End   |
| User Specific | Yes           | Yes           |

---

# 12. Scope Initialization

## Eager Initialization

Created during startup.

Example:

```java
singleton
```

---

## Lazy Initialization

Created when needed.

Examples:

```java
prototype
request
session
```

---

# 13. Scoped Proxy Problem

## Scenario

Request Bean:

```java
@Component
@Scope("request")
public class User {
}
```

Singleton Controller:

```java
@RestController
public class TestController {

    @Autowired
    User user;
}
```

---

## What Happens?

Application Startup:

```text
Create Singleton Controller
      ↓
Need User Bean
      ↓
User Bean requires Request Context
      ↓
No Request Available
```

---

## Error

```text
UnsatisfiedDependencyException
```

Application fails.

---

# 14. Scoped Proxy Solution

## Fix

Use Proxy Mode.

```java
@Component
@Scope(
    value = "request",
    proxyMode =
    ScopedProxyMode.TARGET_CLASS
)
public class User {
}
```

---

## What Happens?

Spring injects:

```text
Proxy Object
```

instead of actual request bean.

---

Runtime:

```text
HTTP Request Arrives
      ↓
Proxy Creates Real User Bean
      ↓
Request Processed
```

---

## Interview Question

What problem occurs when Singleton injects Request Scope Bean?

Answer:

Singleton bean is created during startup, but Request bean requires HTTP request context.

This causes:

```text
UnsatisfiedDependencyException
```

Solution:

```java
proxyMode =
ScopedProxyMode.TARGET_CLASS
```

---

# 15. Injecting Short-Lived Beans into Long-Lived Beans

Problem:

```text
Singleton
     ↓
Request Bean
```

Lifetime mismatch.

---

Solutions:

### Scoped Proxy

```java
proxyMode =
ScopedProxyMode.TARGET_CLASS
```

---

### ObjectProvider

```java
@Autowired
ObjectProvider<User> provider;
```

---

### Provider

```java
@Inject
Provider<User> provider;
```

---

# 16. Session Management and Logout

## Session Scope Controller

```java
@RestController
@Scope("session")
@RequestMapping("/api")
public class TestController {
}
```

---

## Logout Example

```java
@GetMapping("/logout")
public ResponseEntity<String>
logout(
        HttpServletRequest request
) {

    HttpSession session =
            request.getSession();

    session.invalidate();

    return ResponseEntity
            .ok("")
            ;
}
```

---

## What Happens?

```text
Session Destroyed
      ↓
Session Scope Beans Destroyed
```

---

# 17. Real Spring Boot Examples

## Singleton Service

```java
@Service
@Scope("singleton")
public class UserService {
}
```

---

## Prototype Processor

```java
@Component
@Scope("prototype")
public class FileProcessor {
}
```

---

## Request Context

```java
@Component
@Scope(
    value = "request",
    proxyMode =
    ScopedProxyMode.TARGET_CLASS
)
public class RequestInfo {
}
```

---

## Shopping Cart

```java
@Component
@Scope("session")
public class ShoppingCart {
}
```

---

# 18. Frequently Asked Interview Questions

## Basics

1. What is Bean Scope?
2. Why do we need Bean Scopes?
3. What is default Bean Scope?

---

## Singleton Scope

4. What is Singleton Scope?
5. Is Singleton Scope default?
6. When is Singleton Bean created?
7. Is Singleton eager or lazy?

---

## Prototype Scope

8. What is Prototype Scope?
9. Difference between Singleton and Prototype?
10. Is Prototype eager or lazy?

---

## Request Scope

11. What is Request Scope?
12. When is Request Bean created?
13. When is Request Bean destroyed?

---

## Session Scope

14. What is Session Scope?
15. When is Session Bean created?
16. How is Session Bean destroyed?

---

## Advanced

17. What is Scoped Proxy?
18. Why do we need Scoped Proxy?
19. What is lifetime mismatch?
20. Why does Singleton fail to inject Request Bean?
21. What is ObjectProvider?
22. What is Provider?
23. Difference between Request and Session Scope?

---

# 19. Final Mental Model

Singleton

```text
Application Start
      ↓
Create Once
      ↓
Use Forever
```

---

Prototype

```text
Request Bean
      ↓
Create New Object
```

Every time.

---

Request Scope

```text
HTTP Request
      ↓
Create Bean
      ↓
Destroy Bean
```

---

Session Scope

```text
User Login
      ↓
Create Session Bean
      ↓
User Logout
      ↓
Destroy Bean
```

---

## One-Line Interview Summary

Spring Bean Scope determines how many instances of a bean are created and how long they live. Singleton creates one shared instance, Prototype creates a new instance per request, Request creates one per HTTP request, and Session creates one per user session.
