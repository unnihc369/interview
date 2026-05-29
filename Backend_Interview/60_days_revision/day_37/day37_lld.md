# Day 37 — LLD: Mediator & Interpreter Patterns

**Topics:** Mediator Pattern · Interpreter Pattern · Chat Room · Rule Engine · Interview Questions

---

# 1. Mediator Pattern

## Intent

Define an object that encapsulates how a set of objects interact. Promotes loose coupling by avoiding direct references between colleagues.

```text
Without Mediator:  A ↔ B, A ↔ C, B ↔ C  (mesh)
With Mediator:     A → M ← B, C         (star)
```

---

# 2. Mediator — Chat Room Example

```java
public interface ChatMediator {
    void sendMessage(String msg, User user);
    void addUser(User user);
}

public class ChatRoom implements ChatMediator {
    private final List<User> users = new ArrayList<>();

    @Override
    public void addUser(User user) {
        users.add(user);
    }

    @Override
    public void sendMessage(String msg, User sender) {
        for (User u : users) {
            if (u != sender) u.receive(msg, sender.getName());
        }
    }
}

public class User {
    private final String name;
    private final ChatMediator mediator;

    public User(String name, ChatMediator mediator) {
        this.name = name;
        this.mediator = mediator;
        mediator.addUser(this);
    }

    public void send(String msg) {
        mediator.sendMessage(msg, this);
    }

    public void receive(String msg, String from) {
        System.out.println(name + " received from " + from + ": " + msg);
    }
}
```

---

# 3. Mediator in Spring / Real Systems

| System | Mediator Role |
|--------|---------------|
| Air traffic control | Coordinates planes, not plane-to-plane |
| Event bus / Mediator service | Decouples microservices |
| Dialog UI | Widgets talk through dialog coordinator |
| Message broker | Kafka/Rabbit as central mediator |

---

# 4. Interpreter Pattern

## Intent

Given a language, define grammar representation and an interpreter that uses the representation to interpret sentences.

```text
Expression tree → evaluate(context)
```

Best when grammar is simple and statements are frequent.

---

# 5. Interpreter — Boolean Rule Example

```java
public interface Expression {
    boolean interpret(Map<String, Boolean> context);
}

public class VariableExpression implements Expression {
    private final String name;
    public VariableExpression(String name) { this.name = name; }

    @Override
    public boolean interpret(Map<String, Boolean> ctx) {
        return Boolean.TRUE.equals(ctx.get(name));
    }
}

public class AndExpression implements Expression {
    private final Expression left, right;
    public AndExpression(Expression l, Expression r) { left = l; right = r; }

    @Override
    public boolean interpret(Map<String, Boolean> ctx) {
        return left.interpret(ctx) && right.interpret(ctx);
    }
}

public class OrExpression implements Expression {
    private final Expression left, right;
    public OrExpression(Expression l, Expression r) { left = l; right = r; }

    @Override
    public boolean interpret(Map<String, Boolean> ctx) {
        return left.interpret(ctx) || right.interpret(ctx);
    }
}
```

Usage:

```java
// premium AND (age > 18)  →  simplified as feature flags
Expression rule = new AndExpression(
    new VariableExpression("isPremium"),
    new VariableExpression("isAdult")
);
boolean eligible = rule.interpret(Map.of("isPremium", true, "isAdult", true));
```

---

# 6. SQL / Regex as Interpreter

```text
SQL parser → AST → execute nodes
Regex NFA → match input
```

Production rule engines often use **ANTLR** parser + visitor instead of manual Interpreter classes.

---

# 7. Mediator vs Observer

| Mediator | Observer |
|----------|----------|
| Central coordinator | Subject notifies many observers |
| Colleagues don't know each other | Observers know subject |
| Bidirectional through hub | Often one-to-many broadcast |

---

# 8. When to Use (Interview)

| Pattern | Use When |
|---------|----------|
| Mediator | Many objects interact complexly; reduce coupling |
| Interpreter | Simple DSL, rules, formulas; grammar stable |

Avoid Interpreter for complex grammars — use parser generators.

---

# 9. Combined LLD — Rule-Based Notification System

```text
NotificationMediator
  ├── receives OrderPlaced event
  ├── asks RuleInterpreter (VIP AND highValue)
  └── dispatches to EmailColleague, SmsColleague, PushColleague
```

---

# Interview Questions

## Q1. Mediator vs Facade?

Facade simplifies subsystem interface for client. Mediator manages **communication between** components.

## Q2. Real example of Mediator in backend?

Message broker, API gateway, or orchestration saga coordinator.

## Q3. Interpreter performance?

Expression trees can be cached; for hot paths compile to bytecode or precompute.

## Q4. Spring `@Controller` mediator?

DispatcherServlet mediates between HTTP handlers — classic Mediator in framework design.

## Q5. When NOT to use Interpreter?

Large grammar, frequent language changes — use ANTLR, Drools, or script engine.

---

# Quick Revision

```text
Mediator = central hub (chat room, event bus); Interpreter = grammar tree evaluate(context).
```

---

*End of Day 37 LLD*
