# Day 15 — Java Nested & Static Nested Classes

**Topics:** Nested Classes · Static Nested Classes · Inner Classes · Interview Questions

---

# 1. What is a Nested Class?

A class defined inside another class.

```java
class Outer {
    class Inner {
        void display() {
            System.out.println("Inner class");
        }
    }
}
```

---

# Why Use Nested Classes?

- **Logical grouping** — helper classes used only by outer class
- **Encapsulation** — hide implementation details
- **Readability** — keep related code together

Common in backend code for:
- Builder pattern
- DTO grouping
- Event handlers

---

# 2. Types of Nested Classes

| Type | Keyword | Can Access Outer Instance? |
|------|---------|---------------------------|
| Static Nested | `static` | No (unless reference passed) |
| Inner (Non-static) | none | Yes |
| Local | inside method | Yes |
| Anonymous | inline | Yes |

---

# 3. Static Nested Class

## Definition

Nested class marked `static`. Does **not** hold implicit reference to outer object.

```java
class Outer {
    private static int count = 0;

    static class StaticNested {
        void printCount() {
            System.out.println(count); // OK — static to static
        }
    }
}
```

---

## Usage

```java
Outer.StaticNested nested = new Outer.StaticNested();
nested.printCount();
```

No outer instance required.

---

## Real-World Example — API Response Wrapper

```java
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private ErrorDetails error;

    public static class ErrorDetails {
        private String code;
        private String message;

        public ErrorDetails(String code, String message) {
            this.code = code;
            this.message = message;
        }
    }
}
```

Interview signal: groups related DTOs without polluting package namespace.

---

# 4. Inner (Non-Static) Class

## Definition

Holds implicit reference to enclosing instance.

```java
class Outer {
    private String name = "Outer";

    class Inner {
        void printName() {
            System.out.println(name); // direct access to outer field
        }
    }
}
```

---

## Usage

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
inner.printName();
```

---

## When to Use

When nested class needs outer instance state.

Example: iterator over custom collection.

---

# 5. Local & Anonymous Classes

## Local Class

Defined inside a method.

```java
void process() {
    class LocalHelper {
        void help() { /* ... */ }
    }
    new LocalHelper().help();
}
```

---

## Anonymous Class

One-time implementation without naming.

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

**Modern alternative:** lambdas for functional interfaces.

---

# 6. Static Nested vs Inner Class

| Feature | Static Nested | Inner Class |
|---------|---------------|-------------|
| Outer instance needed? | No | Yes |
| Memory | Lighter | Holds outer ref |
| Access outer fields | Only static fields | All fields |
| Common use | DTOs, builders | Iterators, callbacks |

---

# 7. Static Keyword in Nested Context

## Static Nested Class Rules

- Can access outer **static** members directly
- Cannot access outer **instance** members without outer reference
- Can be instantiated without outer object

```java
class ConnectionPool {
    private static int maxSize = 10;

    static class Config {
        int getMaxSize() {
            return maxSize;
        }
    }
}
```

---

# 8. Interview Code Example — Builder Pattern

```java
public class User {
    private final String name;
    private final String email;

    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
    }

    public static class Builder {
        private String name;
        private String email;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}
```

Usage:

```java
User user = new User.Builder()
        .name("Alice")
        .email("alice@example.com")
        .build();
```

---

# Interview Questions

## Difference between static nested and inner class?

Static nested class does not require outer instance and cannot access outer instance fields directly. Inner class requires outer object and can access all outer members.

---

## Can static nested class access outer private members?

**Yes.** Nested classes (static or inner) can access outer class private members. Java treats them as members of the outer class.

---

## Why prefer static nested for DTOs?

Avoids accidental memory leak from holding outer reference. DTOs typically don't need outer state.

---

## Inner class vs separate top-level class?

Use nested when class is tightly coupled and not reused elsewhere. Use top-level when it's a shared domain concept.

---

## Memory leak risk with inner classes?

Yes. Long-lived inner class holding reference to short-lived outer can prevent GC of outer. Prefer static nested when outer reference not needed.

---

# Quick Revision

```text
Static Nested → no outer instance, DTOs/builders
Inner Class   → needs outer instance, iterators
Local/Anonymous → method-scoped, one-off logic
Static nested preferred for API response grouping
```

---

*End of Day 15 Java Nested Classes*
