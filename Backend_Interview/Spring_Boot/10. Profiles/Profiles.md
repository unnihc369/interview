# Spring Boot Profiles & @Profile Annotation — Complete Interview Guide

## Table of Contents

1. Introduction to Profiles
2. Why Profiles Are Needed
3. Environment Separation
4. What is @Profile?
5. How Spring Profiles Work
6. application.properties
7. Profile-Specific Property Files
8. Naming Convention
9. Activating Profiles
10. Using spring.profiles.active
11. Activating Profiles via Command Line
12. Activating Profiles via Maven Profiles
13. @Profile Annotation Examples
14. Multiple Active Profiles
15. Profile Resolution Order
16. Real-World Examples
17. @Profile vs @ConditionalOnProperty
18. Best Practices
19. Common Interview Questions
20. Final Mental Model

---

# 1. Introduction to Profiles

## What are Spring Profiles?

Spring Profiles provide a mechanism for separating configuration and bean definitions based on environments.

Example:

```text
Development Environment
QA Environment
Production Environment
```

Each environment requires different configurations.

---

## Interview Definition

A Profile is a logical grouping mechanism that allows Spring Boot to load different configurations and beans depending on the active environment.

---

# 2. Why Profiles Are Needed

Consider an application deployed in three environments:

### Development

```text
Database URL:
localhost:3306

Username:
devuser

Password:
devpassword
```

---

### QA

```text
Database URL:
qa-db.company.com

Username:
qauser

Password:
qapassword
```

---

### Production

```text
Database URL:
prod-db.company.com

Username:
produser

Password:
prodpassword
```

---

Without Profiles:

```text
One application.properties
Huge if-else logic
Difficult maintenance
```

---

With Profiles:

```text
application-dev.properties
application-qa.properties
application-prod.properties
```

Spring automatically loads the correct configuration.

---

# 3. Environment Separation

Profiles are intended for:

```text
Environment Separation
```

Not:

```text
Business Logic Selection
```

---

Good Use Cases:

```text
Development
QA
Staging
Production
```

---

Bad Use Cases:

```text
OnlineOrder
OfflineOrder
```

Use:

```java
@ConditionalOnProperty
```

instead.

---

## Interview Question

What is the primary intended use of @Profile?

Answer:

Environment separation and environment-specific bean configuration.

---

# 4. What is @Profile?

## Definition

`@Profile` is a Spring annotation that tells Spring to create a bean only when specific profiles are active.

---

## Example

```java
@Profile("dev")
@Component
public class DevDatabaseConfig {
}
```

Bean created only if:

```properties
spring.profiles.active=dev
```

---

# 5. How Spring Profiles Work

Application Starts

↓

Read Active Profile

↓

Load Profile Properties

↓

Evaluate @Profile

↓

Create Matching Beans

↓

Skip Non-Matching Beans

↓

Application Ready

---

# 6. application.properties

Default Configuration File.

Example:

```properties
username=defaultUsername
password=defaultPassword

spring.profiles.active=dev
```

---

## Purpose

Acts as:

```text
Default Configuration
```

for all environments.

---

# 7. Profile-Specific Property Files

Spring Boot supports:

```text
application-dev.properties
application-qa.properties
application-prod.properties
```

Each file overrides defaults.

---

Example:

### application-dev.properties

```properties
username=devuser
password=devpassword

server.port=8080
```

---

### application-qa.properties

```properties
username=qauser
password=qapassword

server.port=8081
```

---

### application-prod.properties

```properties
username=produser
password=prodpassword

server.port=9090
```

---

# 8. Naming Convention

## Standard Convention

```text
application-{profile}.properties
```

Examples:

```text
application-dev.properties
application-qa.properties
application-prod.properties
application-stage.properties
```

---

## YAML Format

```text
application-dev.yml
application-prod.yml
```

also supported.

---

## Interview Question

What is the naming convention for environment-specific configuration files?

Answer:

```text
application-{profile}.properties
```

---

# 9. Activating Profiles

Spring determines active profiles using:

```properties
spring.profiles.active
```

---

Example:

```properties
spring.profiles.active=dev
```

---

Spring loads:

```text
application.properties
+
application-dev.properties
```

---

# 10. Using spring.profiles.active

## application.properties

```properties
spring.profiles.active=prod
```

---

Result:

```text
Load application-prod.properties
```

during startup.

---

# 11. Activating Profiles via Command Line

## Method 1

Run Application:

```bash
mvn spring-boot:run \
-Dspring-boot.run.profiles=prod
```

---

Result:

```text
Production Profile Active
```

---

## Method 2

Executable Jar:

```bash
java -jar app.jar \
--spring.profiles.active=prod
```

---

## Interview Question

How can profiles be dynamically activated?

Answer:

Using:

```bash
mvn spring-boot:run \
-Dspring-boot.run.profiles=prod
```

or

```bash
java -jar app.jar \
--spring.profiles.active=prod
```

---

# 12. Activating Profiles via Maven Profiles

## pom.xml

```xml
<profiles>

    <profile>

        <id>local</id>

        <properties>

            <spring-boot.run.profiles>
                dev
            </spring-boot.run.profiles>

        </properties>

    </profile>

</profiles>
```

---

## Run

```bash
mvn spring-boot:run -Plocal
```

---

Result:

```text
dev profile activated
```

---

# 13. @Profile Annotation Examples

## Development Bean

```java
@Component
@Profile("dev")
public class DevDataSource {

    public DevDataSource() {

        System.out.println(
            "Dev Database Created"
        );
    }
}
```

---

## QA Bean

```java
@Component
@Profile("qa")
public class QaDataSource {

    public QaDataSource() {

        System.out.println(
            "QA Database Created"
        );
    }
}
```

---

## Production Bean

```java
@Component
@Profile("prod")
public class ProdDataSource {

    public ProdDataSource() {

        System.out.println(
            "Production Database Created"
        );
    }
}
```

---

## Startup Example

```properties
spring.profiles.active=qa
```

Output:

```text
QA Database Created
```

Only.

---

# 14. Multiple Active Profiles

Spring allows:

```properties
spring.profiles.active=prod,qa
```

---

Meaning:

```text
Production Beans
+
QA Beans
```

loaded together.

---

## Example

```java
@Profile({"qa","prod"})
```

Bean created if:

```text
qa active
OR
prod active
```

---

# 15. Profile Resolution Order

Spring Loads:

```text
application.properties
        ↓
application-dev.properties
        ↓
Overrides Default Values
```

---

Example:

Default:

```properties
server.port=8080
```

---

Dev:

```properties
server.port=9090
```

---

Final Value:

```text
9090
```

---

# 16. Real-World Examples

## Database Configuration

Dev:

```properties
db.url=localhost
```

Prod:

```properties
db.url=prod-db
```

---

## Logging

Dev:

```properties
logging.level.root=DEBUG
```

Prod:

```properties
logging.level.root=ERROR
```

---

## API Endpoints

Dev:

```properties
payment.url=http://localhost
```

Prod:

```properties
payment.url=https://payment.company.com
```

---

## Retry Values

Dev:

```properties
retry.count=1
```

Prod:

```properties
retry.count=5
```

---

# 17. @Profile vs @ConditionalOnProperty

| Feature           | @Profile               | @ConditionalOnProperty  |
| ----------------- | ---------------------- | ----------------------- |
| Purpose           | Environment Separation | Feature Toggle          |
| Typical Use       | Dev / QA / Prod        | Enable/Disable Features |
| Property Driven   | Indirectly             | Directly                |
| Bean Creation     | Conditional            | Conditional             |
| Environment Based | Yes                    | No                      |

---

## Example

Environment:

```java
@Profile("prod")
```

---

Feature:

```java
@ConditionalOnProperty(
    prefix="payment",
    name="enabled"
)
```

---

# 18. Best Practices

## Use Profiles For

✔ Environment Configuration

✔ Database URLs

✔ Credentials

✔ Logging Levels

✔ Timeouts

✔ API Endpoints

---

## Avoid Using Profiles For

✘ Business Logic

✘ Feature Toggles

✘ Payment Provider Selection

Use:

```java
@ConditionalOnProperty
```

instead.

---

# 19. Frequently Asked Interview Questions

## Basics

1. What is a Spring Profile?
2. Why do we need Profiles?
3. What problem do Profiles solve?

---

## Configuration

4. What is spring.profiles.active?
5. What is the naming convention of profile files?
6. How are profile properties loaded?

---

## @Profile

7. What is @Profile?
8. When is @Profile evaluated?
9. Can multiple profiles be active?

---

## Dynamic Activation

10. How can profiles be activated via command line?
11. How can profiles be activated via Maven?
12. How can profiles be activated via environment variables?

---

## Advanced

13. Difference between @Profile and @ConditionalOnProperty?
14. What happens if no profile is active?
15. Which properties override default values?
16. Can a bean belong to multiple profiles?

---

# 20. Final Mental Model

Application Start

↓

Read spring.profiles.active

↓

Load application.properties

↓

Load application-{profile}.properties

↓

Override Default Values

↓

Evaluate @Profile

↓

Create Matching Beans

↓

Skip Non-Matching Beans

↓

Application Ready

---

## One-Line Interview Summary

Spring Profiles provide environment-specific configuration and bean management by allowing Spring Boot to load different property files and beans for Dev, QA, Stage, and Production environments using the @Profile annotation and spring.profiles.active setting.
