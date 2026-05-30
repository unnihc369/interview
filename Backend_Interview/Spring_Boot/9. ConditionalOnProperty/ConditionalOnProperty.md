# @ConditionalOnProperty — Complete Spring Boot Interview Guide

## Table of Contents

1. Introduction
2. Why @ConditionalOnProperty?
3. What Problem Does It Solve?
4. Understanding Conditional Bean Creation
5. @ConditionalOnProperty Syntax
6. Annotation Attributes Explained
7. MySQL Connection Example
8. NoSQL Connection Example
9. DBConnection Manager Example
10. @Autowired(required = false)
11. @PostConstruct and Conditional Beans
12. application.properties Configuration
13. Real Startup Scenarios
14. Feature Toggle Pattern
15. Advantages
16. Disadvantages
17. @ConditionalOnProperty vs @Profile
18. @ConditionalOnProperty vs @Qualifier
19. Common Real-World Use Cases
20. Frequently Asked Interview Questions
21. Final Mental Model

---

# 1. Introduction

## What is @ConditionalOnProperty?

`@ConditionalOnProperty` is a Spring Boot annotation that allows a bean to be created conditionally based on a property value.

Instead of always creating a bean:

```java
@Component
public class MySQLConnection {
}
```

Spring creates the bean only if a specific condition is satisfied.

---

## Interview Definition

@ConditionalOnProperty allows conditional bean registration based on values defined in application.properties or application.yml.

---

# 2. Why @ConditionalOnProperty?

Imagine a common codebase used by multiple applications.

Application 1:

```text
MySQLConnection
NoSQLConnection
```

---

Application 2:

```text
MySQLConnection
```

Only.

---

Without conditional loading:

```text
Application Startup
        ↓
Create MySQLConnection
Create NoSQLConnection
        ↓
Unused Beans Exist
```

Memory wasted.

Startup time increases.

---

# 3. What Problem Does It Solve?

Traditional Spring:

```java
@Component
public class MySQLConnection {
}
```

Bean is always created.

---

Even if:

```text
Application never uses MySQL
```

the bean still exists.

---

Using:

```java
@ConditionalOnProperty
```

Spring decides:

```text
Create Bean?
      ↓
YES → Create
NO  → Skip
```

---

# 4. Understanding Conditional Bean Creation

Normal Bean Creation:

```text
Application Start
      ↓
Component Scan
      ↓
Create Bean
```

---

Conditional Bean Creation:

```text
Application Start
      ↓
Read Property
      ↓
Condition Check
      ↓
Create Bean OR Skip Bean
```

---

# 5. @ConditionalOnProperty Syntax

## General Syntax

```java
@ConditionalOnProperty(
    prefix = "",
    name = "",
    havingValue = "",
    matchIfMissing = false
)
```

---

## Example

```java
@ConditionalOnProperty(
    prefix = "sqlconnection",
    name = "enabled",
    havingValue = "true"
)
```

Property:

```properties
sqlconnection.enabled=true
```

Bean Created.

---

# 6. Annotation Attributes Explained

## prefix

Represents property group.

Example:

```properties
sqlconnection.enabled=true
```

Prefix:

```java
prefix = "sqlconnection"
```

---

## name

Property key.

```java
name = "enabled"
```

---

## havingValue

Expected value.

```java
havingValue = "true"
```

---

## matchIfMissing

Determines behavior when property is absent.

### false

```java
matchIfMissing = false
```

Property missing:

```text
Bean NOT created
```

---

### true

```java
matchIfMissing = true
```

Property missing:

```text
Bean created
```

---

# 7. MySQL Connection Example

## Bean Definition

```java
@Component

@ConditionalOnProperty(
        prefix = "sqlconnection",
        name = "enabled",
        havingValue = "true",
        matchIfMissing = false
)
public class MySQLConnection {

    public MySQLConnection() {

        System.out.println(
                "Initialization of MySQLConnection Bean"
        );
    }
}
```

---

## application.properties

```properties
sqlconnection.enabled=true
```

---

## Output

```text
Initialization of MySQLConnection Bean
```

Bean Created.

---

# 8. NoSQL Connection Example

```java
@Component

@ConditionalOnProperty(
        prefix = "nosqlconnection",
        name = "enabled",
        havingValue = "true",
        matchIfMissing = false
)
public class NoSQLConnection {

    public NoSQLConnection() {

        System.out.println(
                "Initialization of NoSQLConnection Bean"
        );
    }
}
```

---

## application.properties

```properties
nosqlconnection.enabled=true
```

---

Output:

```text
Initialization of NoSQLConnection Bean
```

---

# 9. DBConnection Manager Example

Suppose application may contain:

```text
MySQLConnection
NoSQLConnection
```

or both.

---

Manager Bean:

```java
@Component
public class DBConnection {

    @Autowired(required = false)
    private MySQLConnection mysqlCon;

    @Autowired(required = false)
    private NoSQLConnection noSqlCon;

    @PostConstruct
    public void init() {

        System.out.println(
                "DB Connection Bean Created"
        );

        System.out.println(
                "Is MySQL null : "
                + (mysqlCon == null)
        );

        System.out.println(
                "Is NoSQL null : "
                + (noSqlCon == null)
        );
    }
}
```

---

# 10. @Autowired(required = false)

## Why Needed?

Without:

```java
@Autowired
private MySQLConnection mysqlCon;
```

Spring expects bean.

---

If bean does not exist:

```text
UnsatisfiedDependencyException
```

occurs.

---

Using:

```java
@Autowired(required = false)
```

Spring allows:

```text
Bean Exists → Inject

Bean Missing → Inject null
```

---

# 11. @PostConstruct and Conditional Beans

## What is @PostConstruct?

Runs exactly once after dependency injection.

---

Example:

```java
@PostConstruct
public void init() {

    System.out.println(
        "Bean Initialized"
    );
}
```

---

Lifecycle:

```text
Create Bean
      ↓
Inject Dependencies
      ↓
@PostConstruct
      ↓
Ready
```

---

# 12. application.properties Configuration

## Enable MySQL

```properties
sqlconnection.enabled=true
nosqlconnection.enabled=false
```

Result:

```text
MySQL Bean Created
NoSQL Bean Skipped
```

---

## Enable NoSQL

```properties
sqlconnection.enabled=false
nosqlconnection.enabled=true
```

Result:

```text
MySQL Bean Skipped
NoSQL Bean Created
```

---

## Enable Both

```properties
sqlconnection.enabled=true
nosqlconnection.enabled=true
```

Result:

```text
MySQL Bean Created
NoSQL Bean Created
```

---

## Disable Both

```properties
sqlconnection.enabled=false
nosqlconnection.enabled=false
```

Result:

```text
No Database Bean Created
```

---

# 13. Real Startup Scenarios

## Scenario 1

Properties:

```properties
sqlconnection.enabled=true
```

Output:

```text
Initialization of MySQLConnection Bean

DB Connection Bean Created

Is MySQL null : false

Is NoSQL null : true
```

---

## Scenario 2

Properties:

```properties
nosqlconnection.enabled=true
```

Output:

```text
Initialization of NoSQLConnection Bean

DB Connection Bean Created

Is MySQL null : true

Is NoSQL null : false
```

---

# 14. Feature Toggle Pattern

One of the biggest uses of:

```java
@ConditionalOnProperty
```

is Feature Toggling.

---

Example:

```properties
payment.new-flow.enabled=true
```

---

Bean:

```java
@Component

@ConditionalOnProperty(
    prefix = "payment.new-flow",
    name = "enabled",
    havingValue = "true"
)
public class NewPaymentFlow {
}
```

---

Enable:

```text
New Feature Active
```

Disable:

```text
Old Feature Active
```

without changing code.

---

# 15. Advantages

## 1. Feature Toggling

Enable/disable features easily.

---

## 2. Memory Optimization

Unused beans not created.

---

## 3. Faster Startup

Fewer beans initialized.

---

## 4. Cleaner Application Context

Only required beans exist.

---

## 5. Better Multi-Tenant Support

Different deployments can use different configurations.

---

## 6. Easier Environment Management

Dev:

```properties
feature.enabled=false
```

Prod:

```properties
feature.enabled=true
```

---

# 16. Disadvantages

## 1. Misconfiguration Risk

Wrong property:

```properties
sqlconnection.enable=true
```

instead of

```properties
sqlconnection.enabled=true
```

Bean never created.

---

## 2. Debugging Complexity

Bean may disappear unexpectedly.

---

## 3. Increased Configuration Management

Many properties to maintain.

---

## 4. Confusing Startup Behavior

Developers may not know why bean is missing.

---

## 5. Overuse Leads To Complexity

Too many conditions:

```text
Hard To Maintain
```

---

# 17. @ConditionalOnProperty vs @Profile

## @Profile

Environment-based.

```java
@Profile("dev")
```

---

Used for:

```text
Development
Testing
Production
```

---

## @ConditionalOnProperty

Property-based.

```java
@ConditionalOnProperty(
    prefix="feature",
    name="enabled"
)
```

---

Used for:

```text
Feature Toggle
Configuration Toggle
```

---

# 18. @ConditionalOnProperty vs @Qualifier

| Feature                | @ConditionalOnProperty | @Qualifier |
| ---------------------- | ---------------------- | ---------- |
| Creates Bean?          | Conditionally          | No         |
| Dependency Resolution? | No                     | Yes        |
| Feature Toggle?        | Yes                    | No         |
| Runtime Config Driven? | Yes                    | No         |

---

# 19. Common Real-World Use Cases

## Database Selection

```text
MySQL
PostgreSQL
MongoDB
```

---

## Payment Gateway

```text
Razorpay
Stripe
PayPal
```

---

## Messaging

```text
Kafka
RabbitMQ
SQS
```

---

## Feature Flags

```text
New UI
New Payment Flow
Beta Features
```

---

## Cloud Providers

```text
AWS
Azure
GCP
```

---

# 20. Frequently Asked Interview Questions

## Basics

1. What is @ConditionalOnProperty?
2. Why do we use it?
3. What problem does it solve?

---

## Annotation Attributes

4. What is prefix?
5. What is name?
6. What is havingValue?
7. What is matchIfMissing?

---

## Bean Lifecycle

8. When does Spring evaluate @ConditionalOnProperty?
9. What happens if property is missing?
10. What happens if condition fails?

---

## Advanced

11. Difference between @Profile and @ConditionalOnProperty?
12. Can multiple conditional beans exist?
13. How does Spring avoid UnsatisfiedDependencyException with optional beans?
14. Why use @Autowired(required=false)?
15. What is Feature Toggle?

---

# 21. Final Mental Model

Application Start

↓

Read application.properties

↓

Evaluate Conditions

↓

@ConditionalOnProperty

↓

Condition True ?

├── Yes → Create Bean

└── No → Skip Bean

↓

Inject Dependencies

↓

@PostConstruct

↓

Application Ready

---

## One-Line Interview Summary

@ConditionalOnProperty is a Spring Boot annotation that conditionally creates beans based on configuration properties, enabling feature toggling, reducing memory usage, improving startup performance, and supporting environment-specific behavior without changing application code.
