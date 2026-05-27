# Day 4 — LLD / Design Patterns

## Topics
- Factory Design Pattern
- Singleton Design Pattern

---

# 1. Factory Design Pattern

## What is Factory Pattern?

Factory Pattern creates objects without exposing creation logic.

---

# Problem Without Factory

```java
Vehicle vehicle = new Car();
```

Client tightly coupled with Car.

---

# Solution

Use factory class.

---

# Step 1: Create Interface

```java
interface Vehicle {
    void start();
}
```

---

# Step 2: Implement Classes

```java
class Car implements Vehicle {

    public void start() {
        System.out.println("Car started");
    }
}

class Bike implements Vehicle {

    public void start() {
        System.out.println("Bike started");
    }
}
```

---

# Step 3: Factory Class

```java
class VehicleFactory {

    public static Vehicle getVehicle(String type) {

        if (type.equals("CAR")) {
            return new Car();
        }

        if (type.equals("BIKE")) {
            return new Bike();
        }

        return null;
    }
}
```

---

# Usage

```java
Vehicle vehicle =
        VehicleFactory.getVehicle("CAR");

vehicle.start();
```

---

# Benefits

- Loose coupling
- Centralized object creation
- Easy maintenance

---

# Real World Example

```text
NotificationFactory
    ↓
EmailNotification
SMSNotification
PushNotification
```

---

# 2. Singleton Design Pattern

## What is Singleton?

Only one object exists throughout application.

---

# Why Needed?

Used for:
- Logger
- Database Connection
- Config Manager

---

# Eager Singleton

```java
class Singleton {

    private static final Singleton instance =
            new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

---

# Lazy Singleton

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

---

# Thread Safe Singleton

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

---

# Double Checked Locking

```java
class Singleton {

    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {

        if (instance == null) {

            synchronized (Singleton.class) {

                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
```

---

# Singleton Using Enum

```java
enum Singleton {
    INSTANCE;
}
```

---

# Interview Questions

## Why constructor private?
To prevent object creation using new.

---

## Why enum singleton preferred?
Thread-safe and prevents reflection/serialization attacks.

---

## Difference between Factory and Singleton?

| Factory | Singleton |
|---|---|
| Creates objects | Restricts object creation |
| Multiple objects | Single object |

---

# Quick Revision

```text
Factory → object creation abstraction
Singleton → only one object
```

---

*End of Day 4 LLD*