# Day 6 — Spring Boot

## Topics
- DTO
- Service Layer
- Repository Layer
- ResponseEntity

---

# DTO

DTO = Data Transfer Object

Used to avoid exposing entities.

---

## Example

```java
class UserDTO {

    private String name;
    private String email;
}
```

---

# Service Layer

Contains business logic.

```java
@Service
class UserService {

    public String getUser() {
        return "User";
    }
}
```

---

# Repository Layer

Handles database operations.

```java
@Repository
interface UserRepository
        extends JpaRepository<User, Long> {

}
```

---

# ResponseEntity

Custom HTTP response.

```java
@GetMapping
public ResponseEntity<String> hello() {

    return ResponseEntity.ok("Hello");
}
```

---

# Layered Architecture

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

---

# Interview Questions

## Why use DTO?
Security and cleaner APIs.

---

## Why service layer?
Separates business logic.

---

# Quick Revision

```text
DTO → API object
Service → business logic
Repository → DB access
```

---

*End of Day 6 Spring Boot*