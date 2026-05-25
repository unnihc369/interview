# Day 2 - SOLID Principles

---

# SOLID Principles

| Letter | Principle             |
| ------ | --------------------- |
| S      | Single Responsibility |
| O      | Open Closed           |
| L      | Liskov Substitution   |
| I      | Interface Segregation |
| D      | Dependency Inversion  |

---

# 1. Liskov Substitution Principle (LSP)

---

# Definition

Objects of child class should replace parent class without breaking behavior.

---

# Bad Example

```java
class Bird {
    void fly() {
        System.out.println("Flying");
    }
}

class Penguin extends Bird {

    void fly() {
        throw new RuntimeException();
    }
}
```

Problem:

Penguin cannot fly.

LSP violated.

---

# Correct Design

```java
interface Bird {
}

interface FlyingBird {
    void fly();
}

class Sparrow implements FlyingBird {

    public void fly() {
        System.out.println("Flying");
    }
}

class Penguin implements Bird {
}
```

---

# Benefits

- Proper inheritance
- Better maintainability
- Prevents runtime issues

---

# Frequently Asked Interview Questions

## Why Penguin Example Famous?

Because it demonstrates wrong inheritance hierarchy.

---

## Main Idea of LSP

Child class should not change expected behavior.

---

# 2. Interface Segregation Principle (ISP)

---

# Definition

Clients should not depend on methods they do not use.

---

# Bad Example

```java
interface Worker {

    void work();

    void eat();
}

class Robot implements Worker {

    public void work() {
        System.out.println("Working");
    }

    public void eat() {
        // unnecessary
    }
}
```

Problem:

Robot does not eat.

ISP violated.

---

# Correct Design

```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class Human implements Workable, Eatable {

    public void work() {
        System.out.println("Working");
    }

    public void eat() {
        System.out.println("Eating");
    }
}

class Robot implements Workable {

    public void work() {
        System.out.println("Robot working");
    }
}
```

---

# Benefits

- Small interfaces
- Flexible design
- Better maintainability

---

# Frequently Asked Interview Questions

## Difference Between LSP and ISP

| LSP                       | ISP                         |
| ------------------------- | --------------------------- |
| Proper inheritance        | Proper interface design     |
| Focus on substitutability | Focus on smaller interfaces |

---

# Real Interview Example

## Payment System

Bad:

```java
interface Payment {
    pay();
    cashback();
}
```

UPI may support cashback.

Bank transfer may not.

---

# Better Design

```java
interface Payment {
    pay();
}

interface Cashback {
    cashback();
}
```

---

# SOLID Interview Questions

## Why SOLID Important?

- Scalable code
- Maintainable systems
- Easier testing
- Flexible architecture

---

## Most Used SOLID Principle?

Usually:

```text
SRP and Dependency Inversion
```

in Spring Boot applications.

---

# Common Follow-Up Questions

## Difference Between Tight and Loose Coupling

| Tight Coupling         | Loose Coupling      |
| ---------------------- | ------------------- |
| Classes depend heavily | Independent classes |
| Hard to change         | Easy to modify      |

---

## Which SOLID Principle Helps Loose Coupling?

```text
Dependency Inversion Principle
```

---

# Quick Revision Notes

| Principle | Main Idea                            |
| --------- | ------------------------------------ |
| LSP       | Child should properly replace parent |
| ISP       | Do not force unused methods          |

```

```
