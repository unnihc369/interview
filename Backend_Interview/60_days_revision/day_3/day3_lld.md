# Day 3 — SOLID Principle (Dependency Inversion Principle)

**Topics:** DIP · Loose Coupling · Dependency Injection · Spring Boot Examples

---

# SOLID Principles Overview

| Letter | Principle |
|---|---|
| S | Single Responsibility |
| O | Open Closed |
| L | Liskov Substitution |
| I | Interface Segregation |
| D | Dependency Inversion |

---

# Dependency Inversion Principle (DIP)

## Definition

> High-level modules should not depend on low-level modules.
Both should depend on abstractions.

---

# Bad Design (Tight Coupling)

```java
class WiredKeyboard {

    public void type() {
        System.out.println("Typing...");
    }
}

class Computer {

    private WiredKeyboard keyboard = new WiredKeyboard();

    public void start() {
        keyboard.type();
    }
}
```

---

# Problems

- Tight coupling
- Hard to replace dependency
- Difficult testing
- Not scalable

---

# Correct Design Using Interface

## Step 1: Create Abstraction

```java
interface Keyboard {
    void type();
}
```

---

## Step 2: Implement Interface

```java
class WiredKeyboard implements Keyboard {

    public void type() {
        System.out.println("Typing using wired keyboard");
    }
}

class BluetoothKeyboard implements Keyboard {

    public void type() {
        System.out.println("Typing using bluetooth keyboard");
    }
}
```

---

## Step 3: Depend on Abstraction

```java
class Computer {

    private Keyboard keyboard;

    public Computer(Keyboard keyboard) {
        this.keyboard = keyboard;
    }

    public void start() {
        keyboard.type();
    }
}
```

---

## Main Method

```java
public class Main {

    public static void main(String[] args) {

        Keyboard keyboard = new BluetoothKeyboard();

        Computer computer = new Computer(keyboard);

        computer.start();
    }
}
```

---

# Output

```text
Typing using bluetooth keyboard
```

---

# Why DIP is Important

- Loose coupling
- Easy maintenance
- Better scalability
- Easier testing
- Flexible architecture

---

# DIP in Spring Boot

Spring Boot internally follows DIP using Dependency Injection.

---

## Example

```java
interface PaymentGateway {
    void pay();
}
```

---

## Implementations

```java
@Service
class RazorpayGateway implements PaymentGateway {

    public void pay() {
        System.out.println("Payment via Razorpay");
    }
}

@Service
class StripeGateway implements PaymentGateway {

    public void pay() {
        System.out.println("Payment via Stripe");
    }
}
```

---

## High Level Module

```java
@Service
class PaymentService {

    private final PaymentGateway gateway;

    public PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    public void processPayment() {
        gateway.pay();
    }
}
```

---

# Why Constructor Injection?

- Immutable dependencies
- Easier testing
- Recommended by Spring

---

# Dependency Injection vs Dependency Inversion

| Dependency Injection | Dependency Inversion |
|---|---|
| Technique | Design Principle |
| Spring feature | SOLID principle |
| Injects dependency | Depends on abstraction |

---

# Real World Example

Food Delivery App:

```text
Swiggy
   ↓
PaymentGateway Interface
   ↓
Razorpay / Stripe / Paytm
```

New payment systems can be added without changing core business logic.

---

# Interview Questions

## What is tight coupling?
When classes directly depend on concrete implementations.

---

## What is loose coupling?
Classes depend on interfaces/abstractions.

---

## Why use interfaces?
For flexibility and extensibility.

---

## How Spring Boot follows DIP?
Using IoC Container and Dependency Injection.

---

# Quick Revision

```text
DIP → Depend on abstraction, not concrete class
DI → Technique used to achieve DIP
Loose Coupling → Flexible architecture
```

---

*End of Day 3 SOLID Principle*