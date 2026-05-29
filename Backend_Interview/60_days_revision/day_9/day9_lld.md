# Day 9 — Decorator Pattern & Command Pattern

**Topics:** Structural vs behavioral · Class diagrams · Sequence flows · Interview points

---

# 1. Decorator Pattern

## Intent

Attach additional responsibilities to an object dynamically without subclass explosion.

```text
Component ← ConcreteComponent
Component ← Decorator wraps Component
```

---

# Class Diagram (Text)

```text
+----------------+
|   Coffee       |<<interface>>
+----------------+
| + cost()       |
| + desc()       |
+----------------+
        △
        |
 +------+--------+
 |               |
+--------+  +-------------+
|Espresso|  |CoffeeDecorator|
+--------+  +-------------+
            | - wrapped     |
            +-------------+
            | + cost()      |
            +-------------+
                    △
            +-------+-------+
            |               |
        +--------+    +--------+
        | Milk   |    | Sugar  |
        +--------+    +--------+
```

---

# Java Implementation

```java
interface Coffee {
    double cost();
    String description();
}

class Espresso implements Coffee {
    public double cost() { return 2.0; }
    public String description() { return "Espresso"; }
}

abstract class CoffeeDecorator implements Coffee {
    protected final Coffee wrapped;
    CoffeeDecorator(Coffee coffee) { this.wrapped = coffee; }
}

class MilkDecorator extends CoffeeDecorator {
    MilkDecorator(Coffee coffee) { super(coffee); }
    public double cost() { return wrapped.cost() + 0.5; }
    public String description() { return wrapped.description() + ", Milk"; }
}

// Usage
Coffee coffee = new MilkDecorator(new Espresso());
System.out.println(coffee.description() + " = $" + coffee.cost());
```

---

# Sequence Flow

```text
Client -> MilkDecorator: cost()
MilkDecorator -> Espresso: cost()
Espresso --> MilkDecorator: 2.0
MilkDecorator --> Client: 2.5
```

---

# Real-World Uses

- Java I/O: `new BufferedInputStream(new FileInputStream("f"))`
- HTTP middleware stacks
- UI component wrapping (border, scroll)

---

# Decorator vs Inheritance

| Subclass per combo | Decorator |
|--------------------|-----------|
| MilkSugarEspresso class explosion | Stack decorators at runtime |
| Compile-time binding | Flexible composition |

---

# 2. Command Pattern

## Intent

Encapsulate a request as an object — supports undo, queue, logging, macro commands.

---

# Class Diagram (Text)

```text
+-------------+
|  Command    |<<interface>>
+-------------+
| + execute() |
| + undo()    |
+-------------+
      △
      |
+-----------+     +-------------+
|LightOnCmd |---->|   Light     | (Receiver)
+-----------+     +-------------+
      |
+-------------+
|  Invoker    |
+-------------+
| - history   |
+-------------+
| + run(cmd)  |
+-------------+
```

---

# Java Implementation

```java
interface Command {
    void execute();
    void undo();
}

class Light {
    private boolean on;
    void on() { on = true; System.out.println("Light ON"); }
    void off() { on = false; System.out.println("Light OFF"); }
}

class LightOnCommand implements Command {
    private final Light light;
    LightOnCommand(Light light) { this.light = light; }
    public void execute() { light.on(); }
    public void undo() { light.off(); }
}

class RemoteControl {
    private final Deque<Command> history = new ArrayDeque<>();

    void press(Command cmd) {
        cmd.execute();
        history.push(cmd);
    }

    void undo() {
        if (!history.isEmpty()) history.pop().undo();
    }
}
```

---

# Sequence Flow — Undo

```text
Client -> RemoteControl: press(LightOnCommand)
RemoteControl -> LightOnCommand: execute()
LightOnCommand -> Light: on()
Client -> RemoteControl: undo()
RemoteControl -> LightOnCommand: undo()
LightOnCommand -> Light: off()
```

---

# Real-World Uses

- Text editor undo/redo
- Transaction rollback steps
- Job queues (each job = command)
- Kafka/event sourcing (command events)

---

# Command vs Strategy (Interview)

| Command | Strategy |
|---------|----------|
| Encapsulates action/request | Encapsulates algorithm |
| Often has undo/receiver | Swaps behavior in context |
| Invoker triggers | Client selects strategy |

---

# Interview Points

1. **Decorator:** Open/Closed — extend behavior without modifying core class.
2. **Command:** Decouple invoker from receiver; enables undo and async execution.
3. **Java example:** `Runnable` is a simple command object.
4. **Pitfall:** Too many small decorator classes — balance with factory.
5. **Spring:** `HandlerInterceptor` chain resembles decorator around request handling.

---

# Quick Revision

```text
Decorator = wrap and add behavior dynamically (I/O streams).
Command = request as object with execute/undo (remote control).
```

---

*End of Day 9 LLD*
