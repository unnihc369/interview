# Day 1 — Core Concepts (OOP · SOLID)

**Topics:** Classes & objects · Inheritance · Polymorphism · SOLID (SRP · OCP)

---

## Table of Contents

1. [OOP — Classes & Objects](#1-oop--classes--objects)
2. [OOP — Inheritance](#2-oop--inheritance)
3. [OOP — Polymorphism](#3-oop--polymorphism)
4. [SOLID — Overview](#4-solid--overview)
5. [Single Responsibility Principle (SRP)](#5-single-responsibility-principle-srp)
6. [Open-Closed Principle (OCP)](#6-open-closed-principle-ocp)
7. [Other SOLID Principles (Quick)](#7-other-solid-principles-quick)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. OOP — Classes & Objects

### Class vs object

| Class | Object |
|-------|--------|
| Blueprint / template | Instance created from class |
| Defined once | Many objects can exist |
| `class Car { }` | `Car myCar = new Car();` |

### Four pillars of OOP

| Pillar | Meaning |
|--------|---------|
| **Encapsulation** | Hide data; expose via methods |
| **Inheritance** | Child class gets parent fields/methods |
| **Polymorphism** | Same interface, different behavior |
| **Abstraction** | Hide complexity; show essentials |

### Encapsulation example (Java)

```java
public class BankAccount {
    private double balance;   // hidden

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

**Why:** Protect `balance` from invalid direct changes (`balance = -1000`).

### Constructor

Runs when object is created.

```java
public class User {
    private String name;

    public User(String name) {
        this.name = name;
    }
}
```

---

## 2. OOP — Inheritance

Child class **extends** parent — reuses and specializes behavior.

```java
public class Animal {
    public void eat() {
        System.out.println("eating");
    }
}

public class Dog extends Animal {
    public void bark() {
        System.out.println("woof");
    }
}

Dog d = new Dog();
d.eat();   // inherited
d.bark();  // own method
```

### `extends` vs `implements`

| `extends` | `implements` |
|-----------|--------------|
| Class → class (single inheritance in Java) | Class → interface |
| IS-A relationship | CAN-DO capability |

```java
public interface Flyable {
    void fly();
}

public class Bird extends Animal implements Flyable {
    @Override
    public void fly() {
        System.out.println("flying");
    }
}
```

### Interview line

> Java supports **single class inheritance** (one parent class) but **multiple interfaces**.

---

## 3. OOP — Polymorphism

**Same method call → different behavior** depending on actual object type.

### Compile-time (method overloading)

Same method name, **different parameters** in same class.

```java
public class MathUtil {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
}
```

### Runtime (method overriding)

Child provides **own implementation** of parent method.

```java
public class Animal {
    public void sound() { System.out.println("some sound"); }
}

public class Dog extends Animal {
    @Override
    public void sound() { System.out.println("bark"); }
}

public class Cat extends Animal {
    @Override
    public void sound() { System.out.println("meow"); }
}

Animal a = new Dog();
a.sound();   // "bark" — runtime type is Dog
```

### Polymorphism with interface

```java
List<String> names = new ArrayList<>();  // interface ref, concrete impl
```

**Interview use:** Service layer depends on `PaymentGateway` interface; swap `StripeGateway` / `PayPalGateway` without changing caller.

---

## 4. SOLID — Overview

| Letter | Principle | One line |
|--------|-----------|----------|
| **S** | Single Responsibility | One class, one reason to change |
| **O** | Open-Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subtypes must replace parent safely |
| **I** | Interface Segregation | Small, focused interfaces |
| **D** | Dependency Inversion | Depend on abstractions, not concrete classes |

Day 1 focus: **SRP** and **OCP** (as requested).

---

## 5. Single Responsibility Principle (SRP)

> A class should have **only one reason to change** — one job.

### Violation

```java
public class UserService {
    public User createUser(User u) { /* DB */ }
    public void sendWelcomeEmail(User u) { /* email */ }
    public String generateReport() { /* PDF */ }
}
```

One class handles: persistence + email + reporting → 3 reasons to change.

### Fix — split responsibilities

```java
public class UserService {
    private final UserRepository repo;
    private final EmailService email;

    public User createUser(User u) {
        User saved = repo.save(u);
        email.sendWelcome(saved);
        return saved;
    }
}

public class UserRepository { /* only DB */ }
public class EmailService { /* only email */ }
```

### Spring Boot mapping

| Layer | Responsibility |
|-------|----------------|
| `Controller` | HTTP only |
| `Service` | Business logic |
| `Repository` | Database access |

### Interview answer

> SRP keeps classes focused, easier to test, and changes in email logic don't force edits in DB code.

---

## 6. Open-Closed Principle (OCP)

> Software entities should be **open for extension**, **closed for modification**.

Add new behavior **without changing** existing working code.

### Violation

```java
public double calculateDiscount(String type, double price) {
    if (type.equals("STUDENT")) return price * 0.9;
    if (type.equals("SENIOR")) return price * 0.85;
    if (type.equals("FESTIVAL")) return price * 0.5;
    return price;
}
```

Every new discount type → **modify** this method (risk breaking old types).

### Fix — strategy + interface (extension)

```java
public interface DiscountStrategy {
    double apply(double price);
}

public class StudentDiscount implements DiscountStrategy {
    public double apply(double price) { return price * 0.9; }
}

public class SeniorDiscount implements DiscountStrategy {
    public double apply(double price) { return price * 0.85; }
}

// NEW type: add new class only — no change to existing strategies
public class FestivalDiscount implements DiscountStrategy {
    public double apply(double price) { return price * 0.5; }
}
```

```java
public class PriceCalculator {
    public double finalPrice(DiscountStrategy strategy, double price) {
        return strategy.apply(price);
    }
}
```

### Spring example

New `NotificationService` implementation:

```java
@Service
public class SmsNotification implements NotificationService {
    @Override
    public void send(String msg) { /* SMS */ }
}
```

Existing code using `NotificationService` interface stays unchanged.

### Interview answer

> OCP = extend via new classes/interfaces/plugins, avoid editing stable core logic.

---

## 7. Other SOLID Principles (Quick)

### L — Liskov Substitution

Subclass must work wherever parent is expected without breaking behavior.

```java
// Bad: Square extends Rectangle but setWidth breaks setHeight assumption
```

### I — Interface Segregation

Don't force classes to implement methods they don't use.

```java
// Bad: one huge Worker interface with cook(), code(), fly()
// Good: Cook, Developer, Pilot separate interfaces
```

### D — Dependency Inversion

High-level modules don't depend on low-level details — both depend on **abstractions**.

```java
// Good — Spring style
@Service
public class OrderService {
    private final PaymentGateway gateway;  // interface
    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

> Connects to `@Autowired` + interfaces in Spring Boot intro.

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| What is OOP? Four pillars? | [§1–§3](#1-oop--classes--objects) |
| Encapsulation example? | [§1](#1-oop--classes--objects) |
| Inheritance vs interface? | [§2](#2-oop--inheritance) |
| Overloading vs overriding? | [§3](#3-oop--polymorphism) |
| What is SRP? | [§5](#5-single-responsibility-principle-srp) |
| What is OCP? | [§6](#6-open-closed-principle-ocp) |
| How Spring layers follow SRP? | [§5](#5-single-responsibility-principle-srp) |
| How to extend without modifying? | [§6](#6-open-closed-principle-ocp) |

### Day 1 cheat sheet

```
Class     → blueprint | Object → instance
extends   → reuse parent | implements → contract
Override  → runtime polymorphism
SRP       → one job per class
OCP       → extend with new classes, don't edit core
```

---

*End of Day 1 concepts*
