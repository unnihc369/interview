# Day 18 — Lambdas & Functional Interfaces

**Topics:** Lambda Expressions · Functional Interfaces · Method References · Stream API Basics

---

# 1. Lambda Expressions

Introduced in Java 8. Anonymous functions — concise way to pass behavior as argument.

```java
// Before Java 8
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda
Runnable r = () -> System.out.println("Hello");
```

---

# Lambda Syntax

```text
(parameters) -> expression
(parameters) -> { statements; }
```

Examples:

```java
(a, b) -> a + b
(x) -> x * 2
() -> System.out.println("Hi")
list.forEach(item -> System.out.println(item));
```

---

# 2. Functional Interfaces

Interface with **exactly one abstract method**. Lambda provides implementation.

```java
@FunctionalInterface
public interface Operation {
    int apply(int a, int b);
}
```

Usage:

```java
Operation add = (a, b) -> a + b;
Operation multiply = (a, b) -> a * b;

System.out.println(add.apply(3, 4));       // 7
System.out.println(multiply.apply(3, 4));  // 12
```

---

# 3. Built-in Functional Interfaces (java.util.function)

| Interface | Method | Use Case |
|-----------|--------|----------|
| `Predicate<T>` | `boolean test(T t)` | Filter conditions |
| `Function<T,R>` | `R apply(T t)` | Transform input to output |
| `Consumer<T>` | `void accept(T t)` | Side effects (print, save) |
| `Supplier<T>` | `T get()` | Lazy/generate values |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | Two-arg transform |
| `UnaryOperator<T>` | `T apply(T t)` | Same-type transform |

---

# Examples

```java
Predicate<String> isEmpty = s -> s == null || s.isBlank();
Function<String, Integer> length = String::length;
Consumer<String> printer = System.out::println;
Supplier<LocalDateTime> now = LocalDateTime::now;

List<String> names = List.of("Alice", "Bob", "Charlie");
names.stream()
     .filter(n -> n.length() > 3)
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

---

# 4. Method References

Shorthand for lambda calling existing method.

| Type | Syntax | Example |
|------|--------|---------|
| Static | `ClassName::method` | `Integer::parseInt` |
| Instance | `instance::method` | `System.out::println` |
| Arbitrary instance | `ClassName::method` | `String::toLowerCase` |
| Constructor | `ClassName::new` | `ArrayList::new` |

```java
// Lambda
list.stream().map(s -> s.toUpperCase())

// Method reference
list.stream().map(String::toUpperCase)
```

---

# 5. @FunctionalInterface Annotation

Optional but recommended. Compiler error if interface has multiple abstract methods.

```java
@FunctionalInterface
public interface Validator<T> {
    boolean validate(T input);

    // default and static methods allowed
    default Validator<T> and(Validator<T> other) {
        return t -> this.validate(t) && other.validate(t);
    }
}
```

---

# 6. Lambdas in Backend Code

## Sorting with Comparator

```java
users.sort(Comparator.comparing(User::getName));
users.sort(Comparator.comparing(User::getAge).reversed());
```

## Event Handlers

```java
button.setOnAction(event -> handleSubmit());
```

## CompletableFuture

```java
CompletableFuture.supplyAsync(() -> fetchFromDb())
    .thenApply(data -> transform(data))
    .thenAccept(result -> cache.put(key, result));
```

## Optional

```java
optionalUser
    .filter(u -> u.isActive())
    .map(User::getEmail)
    .orElse("unknown@example.com");
```

---

# 7. Variable Capture Rules

Lambdas can access:
- Local variables (must be **effectively final**)
- Instance variables
- Static variables

```java
int threshold = 10; // effectively final
list.stream().filter(n -> n > threshold); // OK

threshold = 20; // breaks effectively final — compile error in lambda
```

---

# 8. Custom Functional Interface Example

```java
@FunctionalInterface
public interface EventHandler<T> {
    void handle(T event);
}

public class OrderService {
    public void processOrder(Order order, EventHandler<Order> onComplete) {
        // process...
        onComplete.handle(order);
    }
}

// Usage
orderService.processOrder(order, o -> notificationService.send(o));
```

---

# Interview Questions

## Lambda vs Anonymous Inner Class?

| Lambda | Anonymous Class |
|--------|-----------------|
| No `this` reference to itself | Has its own `this` |
| Cannot shadow variables | Can shadow |
| Only for functional interfaces | Any interface/class |
| Less bytecode | Generates .class file |

---

## What is SAM (Single Abstract Method)?

Functional interface's one abstract method. Lambda implements this method.

---

## Can functional interface have multiple methods?

Yes, if others are `default` or `static`. Only one abstract method required.

---

## Predicate vs Function difference?

`Predicate<T>` returns `boolean` (test/filter). `Function<T,R>` returns transformed value (map).

---

## When NOT to use lambdas?

- Complex multi-statement logic → extract to named method
- Need to reference `this` of enclosing anonymous class
- Debugging stack traces harder with lambdas

---

# Quick Revision

```text
Lambda → (params) -> expression
Functional Interface → single abstract method
Predicate  → filter (test)
Function   → transform (apply)
Consumer   → side effect (accept)
Supplier   → generate (get)
Method Reference → ClassName::method shorthand
Effectively final → required for captured local vars
```

---

*End of Day 18 Lambdas & Functional Interfaces*
