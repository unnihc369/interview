# Day 2 - OOP Concepts

---

# 1. Abstraction

---

# Definition

Abstraction means:

> Hiding implementation details and showing only essential functionality.

---

# Real-Life Example

Car:

- Start()
- Brake()

User does not know engine internals.

---

# Achieved Using

- Abstract Classes
- Interfaces

---

# Abstract Class

## Features

- Can have abstract methods
- Can have concrete methods
- Supports constructors
- Supports state

---

# Java Example

```java
abstract class Vehicle {

    abstract void start();

    void stop() {
        System.out.println("Vehicle stopped");
    }
}

class Car extends Vehicle {

    void start() {
        System.out.println("Car started");
    }
}
```

---

# Interface

## Features

- 100% abstraction (before Java 8)
- Only behavior contract
- Multiple inheritance possible

---

# Java Example

```java
interface Animal {
    void sound();
}

class Dog implements Animal {

    public void sound() {
        System.out.println("Bark");
    }
}
```

---

# Interface vs Abstract Class

| Interface             | Abstract Class             |
| --------------------- | -------------------------- |
| Multiple inheritance  | Single inheritance         |
| Only contract         | Contract + implementation  |
| No constructors       | Constructors allowed       |
| No instance variables | Instance variables allowed |

---

# Frequently Asked Interview Questions

## When to use Interface?

When unrelated classes share behavior.

Example:

```text
Flyable
Runnable
Serializable
```

---

## When to use Abstract Class?

When classes share common code/state.

---

# 2. Encapsulation

---

# Definition

Binding data and methods together while restricting direct access.

---

# Achieved Using

- Private variables
- Getters/Setters

---

# Example

```java
class Employee {

    private int salary;

    public int getSalary() {
        return salary;
    }

    public void setSalary(int salary) {

        if(salary > 0) {
            this.salary = salary;
        }
    }
}
```

---

# Benefits

- Data security
- Controlled access
- Better maintainability

---

# Why Private Variables?

To prevent invalid modification.

---

# Interview Questions

## Difference Between Abstraction and Encapsulation

| Abstraction                       | Encapsulation                   |
| --------------------------------- | ------------------------------- |
| Hides implementation              | Hides data                      |
| Focus on behavior                 | Focus on security               |
| Achieved using abstract/interface | Achieved using access modifiers |

---

# Important OOP Questions

## Can Interface Have Methods?

✅ Yes

Java 8:

- default methods
- static methods

Java 9:

- private methods

---

## Can Abstract Class Have Constructor?

✅ Yes

Used during object initialization.

---

## Can We Create Object of Abstract Class?

❌ No

---

# Real Interview Examples

## Payment Gateway

Interface:

```java
interface Payment {
    void pay();
}
```

Implementations:

- Razorpay
- Stripe
- PayPal

---

# Design Principle

## Program to Interface, not Implementation

Good:

```java
Payment p = new Razorpay();
```

Bad:

```java
Razorpay p = new Razorpay();
```
