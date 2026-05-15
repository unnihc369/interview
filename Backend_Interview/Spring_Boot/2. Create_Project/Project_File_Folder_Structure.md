# Spring Boot Initializr + Layered Architecture (Complete Guide)

## Table of Contents
1. [What is Spring Initializr?](#1-what-is-spring-initializr)
2. [Create Spring Boot Project](#2-create-spring-boot-project)
3. [WAR vs JAR](#3-war-vs-jar)
4. [Generated Project Structure](#4-generated-project-structure)
5. [Layered Architecture](#5-layered-architecture)
6. [Flow of Request](#6-flow-of-request)
7. [Example Project](#7-example-project)
8. [pom.xml](#8-pomxml)
9. [application.properties](#9-applicationproperties)
10. [Entity Layer](#10-entity-layer)
11. [DTO Layer](#11-dto-layer)
12. [Repository Layer](#12-repository-layer)
13. [Service Layer](#13-service-layer)
14. [Controller Layer](#14-controller-layer)
15. [Utility Layer](#15-utility-layer)
16. [Configuration Layer](#16-configuration-layer)
17. [Important Spring Boot Annotations](#17-important-spring-boot-annotations)
18. [Main Class](#18-main-class)
19. [Complete Request Lifecycle](#19-complete-request-lifecycle)
20. [Interview Important Points](#20-interview-important-points)

---

## 1. What is Spring Initializr?

**Spring Initializr** is the official website ([start.spring.io](https://start.spring.io)) used to generate a Spring Boot project with:

- Dependencies
- Build tool (Maven / Gradle)
- Spring Boot version
- Project structure

---

## 2. Create Spring Boot Project

Open [https://start.spring.io](https://start.spring.io) and choose the following:

| Option       | Value                  |
|--------------|------------------------|
| Project      | Maven                  |
| Language     | Java                   |
| Spring Boot  | Latest Stable Version  |
| Group        | `com.example`          |
| Artifact     | `layered-demo`         |
| Name         | `layered-demo`         |
| Packaging    | Jar                    |
| Java         | 17 or 21               |

### Dependencies

- Spring Web
- Spring Data JPA
- Lombok
- MySQL Driver

---

## 3. WAR vs JAR

### JAR (Java Archive)

**Used for:**
- Standalone Spring Boot applications
- Embedded Tomcat server inside app

**Run directly:**
```bash
java -jar app.jar
```

**Advantages:**
- Easy deployment
- Most commonly used
- Embedded server
- Microservices use JAR

**Build command:**
```bash
mvn clean package
```
Produces: `app.jar`

### WAR (Web Archive)

**Used for:**
- Deploying into external servers
- External Tomcat / WebLogic / JBoss

**Example:** `app.war`

**Deploy inside:**
- Apache Tomcat
- WebSphere
- JBoss

### Difference Between JAR and WAR

| Feature              | JAR              | WAR                       |
|----------------------|------------------|---------------------------|
| Server               | Embedded         | External                  |
| Deployment           | Easy             | Complex                   |
| Mostly Used In       | Microservices    | Enterprise legacy apps    |
| Spring Boot Default  | Yes              | No                        |
| Run Command          | `java -jar`      | Deploy to server          |

---

## 4. Generated Project Structure

```
layered-demo
 ├── src/main/java
 │    └── com/example/layereddemo
 │          ├── controller
 │          ├── dto
 │          ├── entity
 │          ├── repository
 │          ├── service
 │          ├── utility
 │          ├── configuration
 │          └── LayeredDemoApplication.java
 │
 ├── src/main/resources
 │      └── application.properties
 │
 └── pom.xml
```

---

## 5. Layered Architecture

### Why Layered Architecture?

It **separates responsibilities** — each layer has a single purpose, making code easier to maintain, test, and scale.

| Layer          | Responsibility                  |
|----------------|---------------------------------|
| Controller     | Handle HTTP requests            |
| DTO            | Transfer data                   |
| Service        | Business logic                  |
| Repository     | Database interaction            |
| Entity         | Database table mapping          |
| Utility        | Common helper methods           |
| Configuration  | App configuration               |

---

## 6. Flow of Request

```
Client
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

Response comes back in reverse order.

---

## 7. Example Project

We will create a **User API** with:

- Save User
- Get User

---

## 8. pom.xml

```xml
<dependencies>

    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- MySQL -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

</dependencies>
```

---

## 9. application.properties

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/testdb
spring.datasource.username=root
spring.datasource.password=root

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

server.port=8080
```

---

## 10. Entity Layer

### `entity/User.java`

```java
package com.example.layereddemo.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;
}
```

---

## 11. DTO Layer

### `dto/UserRequestDto.java`

```java
package com.example.layereddemo.dto;

import lombok.Data;

@Data
public class UserRequestDto {

    private String name;
    private String email;
}
```

### `dto/UserResponseDto.java`

```java
package com.example.layereddemo.dto;

import lombok.Builder;
import lombok.Data;

@Data
@Builder
public class UserResponseDto {

    private Long id;
    private String name;
    private String email;
}
```

### Why DTO is Used?

DTO avoids exposing:

- Database entity directly
- Sensitive fields
- Internal implementation

**Good practice flow:**

```
Controller ↔ DTO ↔ Service ↔ Entity ↔ Repository
```

---

## 12. Repository Layer

### `repository/UserRepository.java`

```java
package com.example.layereddemo.repository;

import com.example.layereddemo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {

}
```

### What Happens Here?

Spring Boot automatically provides:

- `save()`
- `findById()`
- `delete()`
- `findAll()`

Because the interface **extends `JpaRepository`**.

---

## 13. Service Layer

### `service/UserService.java`

```java
package com.example.layereddemo.service;

import com.example.layereddemo.dto.UserRequestDto;
import com.example.layereddemo.dto.UserResponseDto;

public interface UserService {

    UserResponseDto createUser(UserRequestDto requestDto);

    UserResponseDto getUser(Long id);
}
```

### `service/impl/UserServiceImpl.java`

```java
package com.example.layereddemo.service.impl;

import com.example.layereddemo.dto.UserRequestDto;
import com.example.layereddemo.dto.UserResponseDto;
import com.example.layereddemo.entity.User;
import com.example.layereddemo.repository.UserRepository;
import com.example.layereddemo.service.UserService;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    @Override
    public UserResponseDto createUser(UserRequestDto requestDto) {

        User user = User.builder()
                .name(requestDto.getName())
                .email(requestDto.getEmail())
                .build();

        User savedUser = userRepository.save(user);

        return UserResponseDto.builder()
                .id(savedUser.getId())
                .name(savedUser.getName())
                .email(savedUser.getEmail())
                .build();
    }

    @Override
    public UserResponseDto getUser(Long id) {

        User user = userRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("User not found"));

        return UserResponseDto.builder()
                .id(user.getId())
                .name(user.getName())
                .email(user.getEmail())
                .build();
    }
}
```

### Why Service Layer?

Service layer contains:

- Business logic
- Validation
- Calculations
- Transaction handling

> Controller should remain thin — only delegate to service.

---

## 14. Controller Layer

### `controller/UserController.java`

```java
package com.example.layereddemo.controller;

import com.example.layereddemo.dto.UserRequestDto;
import com.example.layereddemo.dto.UserResponseDto;
import com.example.layereddemo.service.UserService;

import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping
    public UserResponseDto createUser(@RequestBody UserRequestDto requestDto) {
        return userService.createUser(requestDto);
    }

    @GetMapping("/{id}")
    public UserResponseDto getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }
}
```

### API Testing

#### Create User

```
POST /users
```

**Request Body:**
```json
{
  "name": "Chinmay",
  "email": "chinmay@gmail.com"
}
```

#### Get User

```
GET /users/1
```

---

## 15. Utility Layer

### `utility/CommonUtil.java`

```java
package com.example.layereddemo.utility;

public class CommonUtil {

    public static String toUpper(String value) {
        return value.toUpperCase();
    }
}
```

### Why Utility Class?

Used for:

- Common reusable logic
- Date conversion
- String formatting
- Validation helpers

---

## 16. Configuration Layer

### `configuration/AppConfig.java`

```java
package com.example.layereddemo.configuration;

import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

}
```

### Why Configuration Class?

Used for:

- Bean creation
- Security config
- Swagger config
- Custom object configuration

**Example:**

```java
@Bean
public ModelMapper modelMapper() {
    return new ModelMapper();
}
```

---

## 17. Important Spring Boot Annotations

| Annotation               | Purpose                  |
|--------------------------|--------------------------|
| `@SpringBootApplication` | Main class               |
| `@RestController`        | REST APIs                |
| `@Service`               | Service layer            |
| `@Repository`            | Repository layer         |
| `@Entity`                | Database table           |
| `@Configuration`         | Configuration class      |
| `@Autowired`             | Dependency Injection     |
| `@RequestBody`           | Read JSON request        |
| `@PathVariable`          | Read URL value           |

---

## 18. Main Class

### `LayeredDemoApplication.java`

```java
package com.example.layereddemo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class LayeredDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(LayeredDemoApplication.class, args);
    }
}
```

---

## 19. Complete Request Lifecycle

### Create User Flow

```
POST /users
    ↓
UserController
    ↓
UserServiceImpl
    ↓
UserRepository
    ↓
MySQL Database
    ↓
Response DTO
```

---

## 20. Interview Important Points

### Why DTO?

- Avoid exposing entity
- Better API design
- Validation support

### Why Interface for Service?

- Loose coupling
- Easier testing
- Multiple implementations possible

### Why Constructor Injection?

Using:

```java
@RequiredArgsConstructor
```

**Advantages:**

- Immutable dependency
- Easier testing
- Recommended by Spring
