# Spring Boot Annotations — Complete Interview Guide

## Table of Contents

1. Java Annotations Fundamentals
2. Reflection in Java
3. How Spring Uses Reflection
4. XML Configuration Era (Before Annotations)
5. Meta Annotations & Composed Annotations
6. @SpringBootApplication
7. @Controller
8. @RestController
9. @ResponseBody
10. @RequestMapping
11. HTTP Method Mapping Annotations
12. @RequestParam
13. @PathVariable
14. @RequestBody
15. ResponseEntity
16. Request Parameter Type Conversion
17. @InitBinder
18. PropertyEditor
19. @ModelAttribute
20. JSON Serialization (Jackson)
21. Dependency Injection Annotations
22. Bean Scopes
23. Stereotype Annotations
24. Configuration Annotations
25. JPA Annotations
26. Frequently Asked Interview Questions

---

# 1. Java Annotations Fundamentals

## What is an Annotation?

An annotation is metadata added to Java code.

```java
@Override
public String toString() {
    return "Hello";
}
```

Annotations by themselves do nothing.

They become useful when:

- Compiler reads them
- Frameworks read them
- Reflection reads them

---

## Common Java Annotations

```java
@Override
@Deprecated
@SuppressWarnings
@FunctionalInterface
```

---

## Retention Policies

### SOURCE

Available only during compilation.

```java
@Override
```

---

### CLASS

Stored in class file.

Not available at runtime.

---

### RUNTIME

Available at runtime.

Used heavily by Spring.

```java
@Retention(RetentionPolicy.RUNTIME)
```

---

# 2. Reflection in Java

## What is Reflection?

Reflection allows Java to inspect classes at runtime.

```java
Class<?> clazz =
        Class.forName(
            "com.example.UserService"
        );
```

Reflection can:

- Read annotations
- Read methods
- Create objects
- Invoke methods

---

## Why Spring Uses Reflection

Spring scans classes and reads:

```java
@Component
@Service
@Repository
@RestController
```

without explicitly creating objects.

---

# 3. How Spring Uses Reflection

Application Starts

↓

Component Scan

↓

Reflection Reads Metadata

↓

Bean Definitions Created

↓

Dependencies Injected

↓

Controllers Registered

---

# 4. XML Configuration Era (Before Annotations)

Old Spring:

```xml
<bean id="userService"
      class="com.example.UserService"/>
```

```xml
<bean id="userController"
      class="com.example.UserController">
</bean>
```

Problems:

- Huge XML files
- Difficult maintenance
- Error-prone

Annotations solved this.

---

# 5. Meta Annotations & Composed Annotations

## What is Meta Annotation?

Annotation applied on another annotation.

Example:

```java
@Controller
```

Internally:

```java
@Component
public @interface Controller {
}
```

---

## Common Composed Annotations

| Annotation             | Internally Contains                                        |
| ---------------------- | ---------------------------------------------------------- |
| @RestController        | @Controller + @ResponseBody                                |
| @GetMapping            | @RequestMapping(GET)                                       |
| @PostMapping           | @RequestMapping(POST)                                      |
| @SpringBootApplication | @Configuration + @EnableAutoConfiguration + @ComponentScan |

---

# 6. @SpringBootApplication

## Purpose

Main entry point of Spring Boot.

```java
@SpringBootApplication
public class App {

    public static void main(String[] args) {

        SpringApplication.run(
                App.class,
                args
        );
    }
}
```

---

## Internally

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

---

# 7. @Controller

## Purpose

Handles web requests.

Returns View Names.

```java
@Controller
@RequestMapping("/pages")
public class PageController {

    @GetMapping("/home")
    public String home() {

        return "home";
    }
}
```

Spring searches:

```text
home.html
home.jsp
```

---

## Meta Annotated With

```java
@Component
```

---

## Interview Question

What is @Controller?

Answer:

A Spring MVC annotation used to mark a class as a web controller. It is meta-annotated with @Component.

---

# 8. @RestController

## Definition

```java
@RestController
```

Internally:

```java
@Controller
@ResponseBody
```

---

## Example

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser() {

        return new User(
            1,
            "John"
        );
    }
}
```

Response:

```json
{
  "id": 1,
  "name": "John"
}
```

---

## Interview Question

What is @RestController?

Answer:

A composed annotation containing:

```java
@Controller
@ResponseBody
```

---

# 9. @ResponseBody

## Purpose

Sends return value directly in HTTP Response Body.

---

### Without @ResponseBody

```java
@Controller
public class DemoController {

    @GetMapping("/hello")
    public String hello() {

        return "hello";
    }
}
```

Spring searches for:

```text
hello.html
hello.jsp
```

---

### With @ResponseBody

```java
@Controller
public class DemoController {

    @GetMapping("/hello")

    @ResponseBody
    public String hello() {

        return "hello";
    }
}
```

Output:

```text
hello
```

---

## Interview Question

What happens if @ResponseBody is not used?

Spring treats returned String as View Name.

---

# 10. @RequestMapping

## Purpose

Maps HTTP Requests.

```java
@RequestMapping(
        value="/users",
        method=RequestMethod.GET
)
```

---

## Attributes

### value/path

URL Mapping

```java
@RequestMapping("/users")
```

---

### method

```java
RequestMethod.GET
RequestMethod.POST
```

---

### consumes

Request Content Type

```java
consumes="application/json"
```

---

### produces

Response Content Type

```java
produces="application/json"
```

---

## Interview Question

Difference between value and path?

No difference.

---

# 11. HTTP Method Mapping Annotations

```java
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

Internally:

```java
@RequestMapping(method=GET)
```

etc.

---

# 12. @RequestParam

## Purpose

Reads Query Parameters.

```java
GET /users?page=1
```

```java
@GetMapping
public String users(
    @RequestParam int page
){
    return "Page " + page;
}
```

---

## Supported Types

- String
- Primitive Types
- Wrapper Classes
- Enum
- List
- Set

---

## Interview Question

How does Spring convert request parameters?

Using:

- ConversionService
- PropertyEditor

---

# 13. @PathVariable

## Purpose

Reads values from URL Path.

```java
/users/10
```

```java
@GetMapping("/users/{id}")
public User getUser(
        @PathVariable Long id
){
    return service.find(id);
}
```

---

## Multiple Variables

```java
@GetMapping(
"/users/{userId}/orders/{orderId}"
)
```

---

## PathVariable vs RequestParam

| PathVariable            | RequestParam    |
| ----------------------- | --------------- |
| URL Path                | Query Parameter |
| Mandatory               | Optional        |
| Resource Identification | Filtering       |

---

# 14. @RequestBody

## Purpose

Maps JSON Request Body to Java Object.

```java
@PostMapping
public User create(
        @RequestBody UserRequest req
){
    return service.save(req);
}
```

Request:

```json
{
  "name": "John",
  "email": "john@gmail.com"
}
```

Spring uses Jackson internally.

---

# 15. ResponseEntity

## Purpose

Represents Complete HTTP Response.

Contains:

- Status Code
- Headers
- Body

---

## Example

```java
@GetMapping("/{id}")
public ResponseEntity<User>
getUser(
        @PathVariable Long id
){

    return ResponseEntity
            .ok(service.find(id));
}
```

---

## Custom Status

```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(user);
```

---

## Interview Question

Difference between Object and ResponseEntity?

Object → Only Body

ResponseEntity → Entire HTTP Response

---

# 16. Request Parameter Type Conversion

Spring automatically converts:

```text
?page=10
```

into:

```java
int page
```

Supported:

- Primitive Types
- Wrapper Classes
- Enum
- Collections
- Dates

---

# 17. @InitBinder

Used before Data Binding.

```java
@InitBinder
public void initBinder(
        WebDataBinder binder
){
}
```

Used for:

- Custom Converters
- String Trimming
- Date Parsing

---

# 18. PropertyEditor

Custom Type Conversion.

Example:

```java
public class MoneyEditor
        extends PropertyEditorSupport {

}
```

Register:

```java
@InitBinder
public void initBinder(
        WebDataBinder binder
){

    binder.registerCustomEditor(
            Money.class,
            new MoneyEditor()
    );
}
```

---

# 19. @ModelAttribute

Maps request parameters to Object.

```java
@GetMapping("/search")
public List<User> search(
        @ModelAttribute SearchDTO dto
){
}
```

Input:

```text
?name=john&age=25
```

becomes:

```java
SearchDTO
```

---

# 20. JSON Serialization (Jackson)

Default JSON Library in Spring Boot.

Used by:

```java
@RequestBody
@ResponseBody
@RestController
```

Annotations:

```java
@JsonProperty
@JsonIgnore
@JsonFormat
```

---

# 21. Dependency Injection Annotations

## @Autowired

```java
@Autowired
private UserService userService;
```

Injects dependency automatically.

---

## Constructor Injection (Recommended)

```java
@Service
public class UserService {

    private final Repo repo;

    public UserService(
            Repo repo
    ){
        this.repo = repo;
    }
}
```

---

## @Qualifier

```java
@Qualifier("smsService")
```

Selects specific bean.

---

## @Primary

Marks default bean.

---

# 22. Bean Scopes

## Singleton (Default)

One bean per container.

```java
@Scope("singleton")
```

---

## Prototype

New object every request.

```java
@Scope("prototype")
```

---

## Request

One bean per HTTP Request.

---

## Session

One bean per Session.

---

# 23. Stereotype Annotations

## @Component

Generic Bean.

---

## @Service

Business Logic Layer.

---

## @Repository

Database Layer.

Provides exception translation.

---

## @Controller

MVC Controller.

---

## @RestController

REST API Controller.

---

# 24. Configuration Annotations

## @Configuration

Configuration Class.

---

## @Bean

Creates Bean manually.

```java
@Bean
public RestTemplate restTemplate() {

    return new RestTemplate();
}
```

---

## @Value

Injects Property.

```java
@Value("${app.name}")
```

---

## @Profile

Environment-specific bean.

```java
@Profile("dev")
```

---

# 25. JPA Annotations

```java
@Entity
@Table
@Id
@GeneratedValue
@Column
```

Relationships:

```java
@OneToMany
@ManyToOne
@OneToOne
@ManyToMany
```

Transaction:

```java
@Transactional
```

---

# 26. Frequently Asked Interview Questions

### Controller Questions

1. What is @Controller?
2. What is @RestController?
3. Difference between @Controller and @RestController?
4. What is @ResponseBody?
5. What happens without @ResponseBody?
6. What is ResponseEntity?

---

### Request Mapping Questions

7. What is @RequestMapping?
8. Difference between @GetMapping and @RequestMapping?
9. What are consumes and produces?
10. Difference between value and path?

---

### Parameter Binding Questions

11. What is @RequestParam?
12. What is @PathVariable?
13. PathVariable vs RequestParam?
14. What is @RequestBody?
15. What is @ModelAttribute?

---

### Spring Core Questions

16. What is @Autowired?
17. Constructor Injection vs Field Injection?
18. What is @Qualifier?
19. What is @Primary?
20. What are Bean Scopes?

---

### Advanced Questions

21. What is Reflection?
22. How does Spring use Reflection?
23. What is PropertyEditor?
24. What is @InitBinder?
25. How does Jackson work?
26. What is @SpringBootApplication?
27. Explain request flow in Spring MVC.
28. How does DispatcherServlet work?

---

# Final Mental Model

Annotations

↓

Reflection

↓

Spring IoC Container

↓

Dependency Injection

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Repository

↓

Database

↓

Response
