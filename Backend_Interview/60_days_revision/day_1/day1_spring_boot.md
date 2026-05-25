# Day 1 — Spring Boot (First REST Controller)

**Topics:** Spring Boot intro · `@RestController` · `@GetMapping` · Request flow · Layered structure

> Deeper background: see `Spring_Boot/1. Into/Introduction_to_spring_boot.md`

---

## Table of Contents

1. [Spring Boot in One Minute](#1-spring-boot-in-one-minute)
2. [Project Structure (Minimal)](#2-project-structure-minimal)
3. [Main Application Class](#3-main-application-class)
4. [First REST Controller](#4-first-rest-controller)
5. [@RestController vs @Controller](#5-restcontroller-vs-controller)
6. [@GetMapping and HTTP Methods](#6-getmapping-and-http-methods)
7. [Request Parameters (Quick)](#7-request-parameters-quick)
8. [Full Request Flow](#8-full-request-flow)
9. [Run and Test](#9-run-and-test)
10. [Interview Quick Index](#10-interview-quick-index)

---

## 1. Spring Boot in One Minute

**Spring Boot** = fast way to build **production-ready** Java apps with:

- Embedded Tomcat (no external server deploy for dev)
- Auto-configuration (`@SpringBootApplication`)
- REST APIs, dependency injection, layered architecture

```
Client  →  HTTP GET /hello  →  Spring Boot app  →  JSON/text response
```

### What you build on Day 1

A single endpoint that returns a message when you hit it in the browser or Postman.

---

## 2. Project Structure (Minimal)

```
my-app/
├── pom.xml
├── src/main/java/com/example/demo/
│   ├── DemoApplication.java          ← entry point
│   └── controller/
│       └── HelloController.java      ← REST API
└── src/main/resources/
    └── application.properties        ← port, config
```

| File | Role |
|------|------|
| `pom.xml` | Dependencies (starter-web) |
| `DemoApplication.java` | Starts Spring + embedded server |
| `HelloController.java` | Your API endpoints |
| `application.properties` | `server.port=8080` etc. |

**Dependency (pom.xml):**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

## 3. Main Application Class

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### What `@SpringBootApplication` does (composed)

| Includes | Purpose |
|----------|---------|
| `@Configuration` | This class can define beans |
| `@EnableAutoConfiguration` | Auto-setup web, Jackson, Tomcat |
| `@ComponentScan` | Scan `com.example.demo` and sub-packages for `@Service`, `@RestController` |

**On startup you see:**

```
Tomcat started on port(s): 8080 (http)
Started DemoApplication in X seconds
```

---

## 4. First REST Controller

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello from Spring Boot!";
    }
}
```

### Test

```
GET http://localhost:8080/hello
```

**Response body:**

```
Hello from Spring Boot!
```

### Return JSON object (common interview extension)

```java
@GetMapping("/user")
public Map<String, Object> getUser() {
    return Map.of(
        "id", 1,
        "name", "Chinmay",
        "role", "SDE"
    );
}
```

Spring converts `Map` / POJO to **JSON** automatically (Jackson).

```java
// Or use a class
public class UserResponse {
    private Long id;
    private String name;
    // getters/setters or Lombok @Data
}

@GetMapping("/user")
public UserResponse getUser() {
    return new UserResponse(1L, "Chinmay");
}
```

---

## 5. @RestController vs @Controller

| `@Controller` | `@RestController` |
|---------------|-------------------|
| Classic MVC | REST API |
| Returns **view name** (HTML page) | Returns **data** in response body |
| Used with Thymeleaf/JSP | Used for JSON APIs |

**`@RestController` = `@Controller` + `@ResponseBody`** on every method.

```java
@Controller
public class PageController {
    @GetMapping("/home")
    public String home() {
        return "home";   // → templates/home.html
    }
}

@RestController
public class ApiController {
    @GetMapping("/api/home")
    public String home() {
        return "home";   // → literal text "home" in HTTP body
    }
}
```

**Interview line:** For backend SDE interviews building APIs, use `@RestController`.

---

## 6. @GetMapping and HTTP Methods

### Class-level + method-level path

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @GetMapping
    public List<String> getAll() {
        return List.of("Alice", "Bob");
    }

    @GetMapping("/{id}")
    public String getOne(@PathVariable Long id) {
        return "User " + id;
    }
}
```

| URL | Handler |
|-----|---------|
| `GET /api/v1/users` | `getAll()` |
| `GET /api/v1/users/42` | `getOne(42)` |

### HTTP method annotations (composed)

| Annotation | HTTP method |
|------------|-------------|
| `@GetMapping` | GET — read |
| `@PostMapping` | POST — create |
| `@PutMapping` | PUT — update |
| `@DeleteMapping` | DELETE — remove |

All are shortcuts for `@RequestMapping(method = RequestMethod.GET)` etc.

### Day 1 scope

Focus on **GET** first. POST/PUT come with `@RequestBody` on later days.

---

## 7. Request Parameters (Quick)

```java
// Query: GET /search?q=java
@GetMapping("/search")
public String search(@RequestParam String q) {
    return "You searched: " + q;
}

// Path: GET /users/10
@GetMapping("/users/{id}")
public String getUser(@PathVariable Long id) {
    return "User id: " + id;
}
```

| Annotation | From |
|------------|------|
| `@RequestParam` | Query string `?key=value` |
| `@PathVariable` | URL path `/users/{id}` |
| `@RequestBody` | JSON body (POST/PUT) |

---

## 8. Full Request Flow

```
Browser: GET http://localhost:8080/hello
        ↓
Embedded Tomcat receives request
        ↓
DispatcherServlet (front controller)
        ↓
Handler mapping finds HelloController.sayHello()
        ↓
Spring creates/injects controller bean
        ↓
Method runs → returns String
        ↓
@ResponseBody (via @RestController) writes to HTTP response
        ↓
Client receives "Hello from Spring Boot!"
```

### How this replaces old Servlet code

| Old Servlet | Spring Boot |
|-------------|-------------|
| `web.xml` URL mapping | `@GetMapping("/hello")` |
| `HttpServlet` + `doGet()` | Controller method |
| Manual response write | Return value → auto-serialized |

---

## 9. Run and Test

### Run

```bash
# IDE: Run DemoApplication.main()
# OR
./mvnw spring-boot:run
```

### Test endpoints

| Tool | Action |
|------|--------|
| Browser | Open `http://localhost:8080/hello` |
| curl | `curl http://localhost:8080/hello` |
| Postman | GET request to same URL |

### Optional — change port

```properties
# application.properties
server.port=9090
```

Then: `http://localhost:9090/hello`

---

## 10. Interview Quick Index

| Question | Answer |
|----------|--------|
| What is `@RestController`? | REST controller; return value → response body (JSON/text) |
| `@RestController` vs `@Controller`? | REST data vs HTML view |
| What is `@GetMapping`? | Maps HTTP GET to method |
| What starts the app? | `@SpringBootApplication` + `SpringApplication.run()` |
| Default server? | Embedded Tomcat |
| Default port? | 8080 |
| How is JSON returned? | Return POJO/Map + Jackson on classpath |
| What is DispatcherServlet? | Front controller — routes request to correct method |
| Where to put API code? | `controller` package |

### Connect to Day 1 concepts

| Topic | Link |
|-------|------|
| SRP | Controller only handles HTTP; logic in Service |
| OCP | New endpoints = new methods/classes, not editing core framework |
| Polymorphism | Inject `Service` interface into controller |

### Day 1 cheat sheet

```
@SpringBootApplication  → start app + component scan
@RestController         → API class, body = return value
@GetMapping("/path")    → HTTP GET handler
GET localhost:8080/path → test in browser
```

---

*End of Day 1 Spring Boot*
