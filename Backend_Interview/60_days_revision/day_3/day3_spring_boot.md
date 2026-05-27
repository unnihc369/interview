# Day 3 — Spring Boot Configuration

**Topics:** application.properties · Profiles · Environment Configurations

---

# 1. application.properties

Used for:
- Server configuration
- Database configuration
- Logging
- Environment setup

Location:
```text
src/main/resources/application.properties
```

---

# Example

```properties
server.port=8081

spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root
```

---

# Common Properties

## Change Port

```properties
server.port=9090
```

---

## Logging

```properties
logging.level.root=DEBUG
```

---

## Context Path

```properties
server.servlet.context-path=/api
```

URL becomes:
```text
localhost:8080/api/users
```

---

# 2. Profiles in Spring Boot

Profiles help maintain different configurations for:
- Development
- Production
- Testing

---

# application-dev.properties

```properties
server.port=8081

spring.datasource.url=jdbc:mysql://localhost/devdb
```

---

# application-prod.properties

```properties
server.port=8080

spring.datasource.url=jdbc:mysql://prodserver/proddb
```

---

# Activate Profile

## Using application.properties

```properties
spring.profiles.active=dev
```

---

## Using VM Options

```text
-Dspring.profiles.active=prod
```

---

## Using Command Line

```bash
java -jar app.jar --spring.profiles.active=prod
```

---

# @Profile Annotation

```java
@Component
@Profile("dev")
public class DevDatabase {
}
```

Bean only loads in dev profile.

---

# Real World Usage

## Dev Environment
- Local DB
- Debug logs
- Testing APIs

## Production Environment
- Secure configs
- Optimized logging
- Real database

---

# application.yml vs application.properties

## application.properties

```properties
server.port=8080
```

---

## application.yml

```yaml
server:
  port: 8080
```

YAML is cleaner for nested configs.

---

# Interview Questions

## Why use profiles?
Different environments need different configurations.

---

## Can multiple profiles be active?
Yes.

```properties
spring.profiles.active=dev,test
```

---

## Which file loads first?
application.properties loads first,
then profile-specific overrides happen.

---

# Quick Revision

```text
application.properties → main config file
Profiles → environment-specific configuration
@Profile → conditional bean loading
```

---

*End of Day 3 Spring Boot*