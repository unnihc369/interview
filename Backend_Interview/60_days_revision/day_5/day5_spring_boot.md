# Day 4 — Spring Boot

## Exception Handling in Spring Boot

Topics:
- @ControllerAdvice
- @ExceptionHandler
- Global exception handling

---

# Why Exception Handling?

Without handling:
- ugly error responses
- stack traces exposed
- inconsistent API response

---

# @ExceptionHandler

Handles specific exception.

---

## Example

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public String getUser() {

        throw new RuntimeException("User not found");
    }

    @ExceptionHandler(RuntimeException.class)
    public String handleException(RuntimeException e) {
        return e.getMessage();
    }
}
```

---

# Global Exception Handling

Use:
```java
@ControllerAdvice
```

---

# Example

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(
            Exception ex) {

        return new ResponseEntity<>(
                ex.getMessage(),
                HttpStatus.INTERNAL_SERVER_ERROR
        );
    }
}
```

---

# Custom Exception Example

```java
class UserNotFoundException
        extends RuntimeException {

    public UserNotFoundException(String message) {
        super(message);
    }
}
```

---

# Handling Custom Exception

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<String> handleUserNotFound(
        UserNotFoundException ex) {

    return new ResponseEntity<>(
            ex.getMessage(),
            HttpStatus.NOT_FOUND
    );
}
```

---

# Standard API Error Response

```json
{
  "message": "User not found",
  "status": 404,
  "timestamp": "2026-05-27"
}
```

---

# Benefits

- Centralized handling
- Cleaner controllers
- Consistent response
- Better API design

---

# Interview Questions

## Difference between @ControllerAdvice and @ExceptionHandler?

@ControllerAdvice:
- global handling

@ExceptionHandler:
- handles specific exception

---

## Why use global exception handling?
To avoid duplicate code.

---

## Which status code for resource not found?
```text
404 NOT FOUND
```

---

# Quick Revision

```text
@ExceptionHandler → handles exception
@ControllerAdvice → global handling
ResponseEntity → custom response
```

---

*End of Day 4 Spring Boot*