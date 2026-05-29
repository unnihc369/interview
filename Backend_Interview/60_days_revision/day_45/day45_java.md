# Day 45 — MethodHandle & invoke

**Topics:** java.lang.invoke · LambdaMetafactory · Performance vs Reflection · Interview Q&A

---

# 1. What is MethodHandle?

Introduced in Java 7 (`java.lang.invoke`). A **typed, directly executable** reference to an underlying method, constructor, or field.

```text
Reflection = inspect/call (slow, checked exceptions, security checks)
MethodHandle = JVM-optimized callable (closer to direct invoke)
```

---

# 2. Obtaining MethodHandles

```java
import java.lang.invoke.*;

public class MethodHandleDemo {

    public String greet(String name) {
        return "Hello, " + name;
    }

    public static void main(String[] args) throws Throwable {
        MethodHandles.Lookup lookup = MethodHandles.lookup();

        // Instance method: greet(String)
        MethodType type = MethodType.methodType(String.class, String.class);
        MethodHandle mh = lookup.findVirtual(
            MethodHandleDemo.class, "greet", type);

        MethodHandleDemo obj = new MethodHandleDemo();
        String result = (String) mh.invoke(obj, "World");
        // "Hello, World"

        // Static method
        MethodHandle staticMh = lookup.findStatic(
            Math.class, "max", MethodType.methodType(int.class, int.class, int.class));
        int max = (int) staticMh.invoke(3, 7);

        // Constructor
        MethodHandle ctor = lookup.findConstructor(
            ArrayList.class, MethodType.methodType(void.class));
        ArrayList<?> list = (ArrayList<?>) ctor.invoke();
    }
}
```

---

# Lookup Permissions

| Lookup | Access |
|--------|--------|
| `MethodHandles.lookup()` | Same module/class as caller |
| `MethodHandles.publicLookup()` | Public methods only |
| `MethodHandles.privateLookupIn()` | Java 9+ module-aware private access |

---

# 3. invoke vs invokeExact

```java
// invoke — converts args/return (boxing, widening)
Object result = mh.invoke(obj, "Alice");

// invokeExact — strict signature match, faster, throws WrongMethodTypeException
String result = (String) mh.invokeExact(obj, "Alice");
```

Use `invokeExact` when types are known at compile time (inside generated code).

---

# 4. MethodHandle Combinators

Chain and adapt handles without reflection.

```java
MethodHandle mh = lookup.findVirtual(String.class, "length",
    MethodType.methodType(int.class));

// Filter return value
MethodHandle twice = MethodHandles.filterReturnValue(mh,
    MethodHandles.constant(int.class, 0)); // always returns 0

// Bind first argument (partial application)
MethodHandle bound = MethodHandles.insertArguments(mh, 0, "hello");
int len = (int) bound.invoke(); // 5

// Fold (combine multiple handles)
MethodHandle sum = lookup.findStatic(Math.class, "addExact",
    MethodType.methodType(int.class, int.class, int.class));
```

---

# 5. MethodHandle vs Reflection

| | Reflection | MethodHandle |
|---|------------|--------------|
| API | `Method.invoke()` | `MethodHandle.invoke()` |
| Type safety | Runtime | `MethodType` enforced |
| Performance | Slower (many checks) | JVM can inline |
| Access | `setAccessible(true)` | `Lookup` capabilities |
| Use case | Frameworks, tools | Lambdas, compilers, DSLs |

---

# 6. Connection to Lambda Expressions

Java 8 lambdas compile to **`invokedynamic`** + **LambdaMetafactory**, not anonymous inner classes.

```java
Function<String, Integer> fn = String::length;
// Bytecode: invokedynamic → LambdaMetafactory.metafactory(...)
// Creates MethodHandle behind the scenes
```

```text
Lambda / Method Reference
        ↓
  LambdaMetafactory
        ↓
  MethodHandle (actual call target)
        ↓
  invokedynamic (runtime linkage)
```

---

# 7. Practical Example: Dynamic Router

```java
public class CommandRouter {
    private final Map<String, MethodHandle> commands = new HashMap<>();
    private final MethodHandles.Lookup lookup = MethodHandles.lookup();

    public void register(String name, Object target, String methodName,
                         MethodType type) throws NoSuchMethodException, IllegalAccessException {
        MethodHandle mh = lookup.findVirtual(target.getClass(), methodName, type);
        mh = mh.bindTo(target);
        commands.put(name, mh);
    }

    public Object dispatch(String name, Object... args) throws Throwable {
        MethodHandle mh = commands.get(name);
        if (mh == null) throw new IllegalArgumentException(name);
        return mh.invokeWithArguments(args);
    }
}
```

---

# 8. VarHandle (Java 9+) — Bonus

Atomic field access without synchronization — sibling of MethodHandle for fields.

```java
VarHandle counterHandle = MethodHandles.lookup()
    .findVarHandle(MyClass.class, "counter", int.class);

MyClass obj = new MyClass();
counterHandle.getAndAdd(obj, 1);  // atomic increment
```

Used in `ConcurrentHashMap`, `Atomic*` internals.

---

# Interview Questions

## Q1. MethodHandle vs Method.invoke?

MethodHandle is type-checked at link time, optimizable by JIT. Reflection validates on every call.

## Q2. Who uses MethodHandle in JDK?

LambdaMetafactory, String concatenation (Java 9+), MethodHandles for `Method`/`VarHandle` in concurrent utilities.

## Q3. Can MethodHandle access private methods?

Yes with appropriate `Lookup` — `privateLookupIn` in Java 9+ modules.

## Q4. invokeWithArguments vs spreadInvoker?

`invokeWithArguments` takes Object array. `asSpreader` adapts varargs calls.

## Q5. When would YOU use MethodHandle?

Building frameworks, expression evaluators, protocol routers — when you need dynamic dispatch with near-direct-call performance. Rare in business CRUD code.

---

# One-Line Revision

```text
MethodHandle = typed, JVM-optimizable method reference; powers lambdas via LambdaMetafactory; faster and safer than raw Reflection.
```

---

*End of Day 45 Java*
