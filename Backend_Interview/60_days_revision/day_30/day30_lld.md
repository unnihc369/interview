# Day 30 — LLD: Flyweight & Bridge Patterns

**Topics:** Structural Patterns · Real Examples · Interview Implementation

---

# Part 1: Flyweight Pattern

---

# 1. Intent

Minimize memory by **sharing** intrinsic (immutable/reusable) state among many similar objects. Extrinsic state passed at runtime.

**Use when:** huge number of similar objects (game tiles, text characters, tree types in forest).

---

# 2. Structure

```text
FlyweightFactory
  └── Map<key, Flyweight> cache

Flyweight (interface)
  └── operation(extrinsicState)

ConcreteFlyweight
  └── intrinsicState (shared)

Client
  └── holds extrinsic state, asks factory for flyweight
```

---

# 3. Example — Forest (Trees)

```java
// Intrinsic (shared)
public class TreeType {
    private final String name;
    private final String color;
    private final String texture; // heavy asset, shared

    public TreeType(String name, String color, String texture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
    }
    public void draw(Canvas canvas, int x, int y) { /* use texture */ }
}

// Flyweight factory
public class TreeFactory {
    private static final Map<String, TreeType> cache = new HashMap<>();

    public static TreeType getTreeType(String name, String color, String texture) {
        String key = name + color + texture;
        return cache.computeIfAbsent(key,
            k -> new TreeType(name, color, texture));
    }
}

// Extrinsic (per-tree position)
public class Tree {
    private final int x, y;
    private final TreeType type;

    public Tree(int x, int y, TreeType type) {
        this.x = x; this.y = y; this.type = type;
    }
    public void draw(Canvas canvas) { type.draw(canvas, x, y); }
}

// Client
public class Forest {
    private final List<Tree> trees = new ArrayList<>();

    public void plant(int x, int y, String name, String color, String texture) {
        TreeType type = TreeFactory.getTreeType(name, color, texture);
        trees.add(new Tree(x, y, type));
    }
}
```

1 million trees, 3 types → 3 `TreeType` objects instead of 1M.

---

# 4. Flyweight in JDK

| Example | Shared (intrinsic) | Extrinsic |
|---------|-------------------|-----------|
| `Integer.valueOf(127)` | cached -128 to 127 | unboxed usage context |
| String constant pool | literal strings | — |
| CSS classes in UI | style definition | DOM element position |

---

# Interview: When NOT to use?

- Few objects — overhead of factory/cache not worth it
- Intrinsic state changes often — breaks sharing
- Extrinsic state dominates memory anyway

---

# Part 2: Bridge Pattern

---

# 5. Intent

**Decouple abstraction from implementation** so both can vary independently. Avoids explosion of subclasses (e.g., Shape×Color matrix).

---

# 6. Structure

```text
Abstraction (remote control)
  └── ref to Implementor

RefinedAbstraction (advanced remote)

Implementor (interface)
  └── ConcreteImplementorA (Sony TV)
  └── ConcreteImplementorB (Samsung TV)
```

---

# 7. Example — Remote Control & Devices

```java
// Implementor
public interface Device {
    void turnOn();
    void turnOff();
    void setVolume(int percent);
}

public class Tv implements Device {
    public void turnOn() { System.out.println("TV on"); }
    public void turnOff() { System.out.println("TV off"); }
    public void setVolume(int p) { System.out.println("TV volume " + p); }
}

public class Radio implements Device {
    public void turnOn() { System.out.println("Radio on"); }
    public void turnOff() { System.out.println("Radio off"); }
    public void setVolume(int p) { System.out.println("Radio volume " + p); }
}

// Abstraction
public abstract class RemoteControl {
    protected Device device;

    protected RemoteControl(Device device) { this.device = device; }

    public void togglePower() {
        // simplified — track state in real impl
        device.turnOn();
    }
}

public class BasicRemote extends RemoteControl {
    public BasicRemote(Device d) { super(d); }
    public void volumeUp() { device.setVolume(50); }
}

public class AdvancedRemote extends RemoteControl {
    public AdvancedRemote(Device d) { super(d); }
    public void mute() { device.setVolume(0); }
}
```

Add new `Device` or new `Remote` without N×M subclass explosion.

---

# 8. Bridge vs Adapter vs Strategy

| Pattern | Purpose |
|---------|---------|
| **Bridge** | Design-time split of abstraction/implementation |
| **Adapter** | Makes incompatible interface work (legacy integration) |
| **Strategy** | Swap algorithms at runtime (behavior only) |

Bridge: "remote controls work with any device" — planned decoupling.  
Strategy: "sort using quicksort or mergesort" — interchangeable algorithm.

---

# 9. Real-World Bridge

```text
JDBC: DriverManager (abstraction) → Driver (implementor) per DB
SLF4J: Logger API → Logback/Log4j binding
Message senders: NotificationService → Email/SMS/Push implementors
```

---

# Interview Questions

## Flyweight vs Singleton?

Singleton = one instance globally. Flyweight = one instance **per distinct intrinsic key** (many shared instances in factory).

## Bridge vs inheritance for Shape×Color?

Inheritance: `RedCircle`, `BlueCircle`, `RedSquare`... Bridge: `Shape` holds `Color` implementor — 2 + 2 classes instead of 4+.

## Thread safety of Flyweight factory?

`ConcurrentHashMap` + immutable flyweights. Double-checked locking if lazy init heavy objects.

---

# Quick Revision

```text
Flyweight: share intrinsic state via factory cache
Bridge: abstraction + implementor vary independently
Bridge ≠ Adapter (integration) ≠ Strategy (algorithm swap)
```

---

*End of Day 30 Flyweight & Bridge LLD*
