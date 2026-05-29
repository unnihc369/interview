# Day 8 — Strategy Pattern & Observer Pattern

**Topics:** Behavioral patterns · Class diagrams · Sequence flows · Interview points

---

# 1. Strategy Pattern

## Intent

Define a family of algorithms, encapsulate each one, and make them interchangeable at runtime.

```text
Context uses Strategy interface — concrete strategy injected at runtime.
```

---

# Problem Without Strategy

```java
class PaymentService {
    void pay(String type, double amount) {
        if ("UPI".equals(type)) { /* UPI logic */ }
        else if ("CARD".equals(type)) { /* card logic */ }
        else if ("WALLET".equals(type)) { /* wallet */ }
    }
}
```

Violates Open/Closed Principle — every new payment type modifies this class.

---

# Class Diagram (Text)

```text
+------------------+
| PaymentContext   |
+------------------+
| - strategy       |
+------------------+
| + pay(amount)    |
+------------------+
        |
        | uses
        v
+------------------+
| PaymentStrategy  |<<interface>>
+------------------+
| + pay(amount)    |
+------------------+
        △
        |
   +----+----+---------+
   |         |         |
+------+ +------+ +--------+
| Upi  | | Card | | Wallet |
+------+ +------+ +--------+
```

---

# Java Implementation

```java
interface PaymentStrategy {
    void pay(double amount);
}

class UpiPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paid " + amount + " via UPI");
    }
}

class CardPayment implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("Paid " + amount + " via Card");
    }
}

class PaymentContext {
    private PaymentStrategy strategy;

    public PaymentContext(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void checkout(double amount) {
        strategy.pay(amount);
    }
}
```

---

# Sequence Flow — Checkout

```text
Client -> PaymentContext: checkout(500)
PaymentContext -> PaymentStrategy: pay(500)
PaymentStrategy --> PaymentContext: success
PaymentContext --> Client: done
```

---

# Real-World Uses

- Sorting comparators (`Comparator` is Strategy)
- Compression (ZIP, GZIP)
- Tax calculation by region
- Shipping cost calculators

---

# 2. Observer Pattern

## Intent

Define one-to-many dependency: when subject state changes, all observers are notified automatically.

---

# Class Diagram (Text)

```text
+----------------+
|   Subject      |
+----------------+
| - observers[]  |
+----------------+
| + attach(o)    |
| + detach(o)    |
| + notify()     |
+----------------+
        |
        | notifies
        v
+----------------+
|   Observer     |<<interface>>
+----------------+
| + update()     |
+----------------+
        △
        |
 +------+------+
 |             |
+--------+  +--------+
| Email  |  |  SMS   |
|Observer|  |Observer|
+--------+  +--------+
```

---

# Java Implementation

```java
interface Observer {
    void update(String event);
}

interface Subject {
    void attach(Observer o);
    void detach(Observer o);
    void notifyObservers();
}

class OrderSubject implements Subject {
    private final List<Observer> observers = new ArrayList<>();

    public void attach(Observer o) { observers.add(o); }
    public void detach(Observer o) { observers.remove(o); }

    public void notifyObservers() {
        for (Observer o : observers) {
            o.update("ORDER_PLACED");
        }
    }

    public void placeOrder() {
        // business logic
        notifyObservers();
    }
}

class EmailNotifier implements Observer {
    public void update(String event) {
        System.out.println("Email: " + event);
    }
}
```

---

# Sequence Flow — Order Placed

```text
Client -> OrderSubject: placeOrder()
OrderSubject -> OrderSubject: notifyObservers()
OrderSubject -> EmailNotifier: update("ORDER_PLACED")
OrderSubject -> SmsNotifier: update("ORDER_PLACED")
EmailNotifier --> OrderSubject: ok
SmsNotifier --> OrderSubject: ok
OrderSubject --> Client: order confirmed
```

---

# Java Built-in Observer (Legacy)

`java.util.Observable` / `Observer` — deprecated in Java 9+. Prefer:

- `PropertyChangeListener`
- Reactive streams (RxJava, Project Reactor)
- Spring `ApplicationEventPublisher`

---

# Strategy vs Observer (Interview)

| Strategy | Observer |
|----------|----------|
| Client chooses algorithm | Subject pushes updates |
| One active strategy | Many observers |
| Behavior variation | Event notification |

---

# Interview Points

1. **When Strategy?** Multiple interchangeable algorithms; avoid giant if-else.
2. **When Observer?** Decouple event source from listeners (notifications, metrics).
3. **SOLID:** Strategy → Open/Closed; Observer → Single Responsibility per listener.
4. **Pitfall:** Observer leak if not detached; memory leaks in long-lived subjects.
5. **Spring example:** `@EventListener` for domain events.

---

# Quick Revision

```text
Strategy = swap algorithm at runtime (payment, sorting).
Observer = broadcast state change to subscribers (order events).
```

---

*End of Day 8 LLD*
