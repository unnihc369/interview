# Day 30 — JVM Classloader & Runtime Areas

**Topics:** Class Loading · Memory Model · Metaspace · Interview Questions

---

# 1. Class Loading Lifecycle

```text
Loading     → read .class binary, create Class object
Linking     → verification → preparation → resolution (optional)
Initialization → execute <clinit>, static fields initialized
```

Class loaded **on first active use** (not merely reference):

- `new` instance
- Static method/field access (declaring class)
- Reflection
- Subclass loaded → parent loaded first
- JVM startup class (main)

---

# 2. Classloader Hierarchy (Delegation Model)

```text
Bootstrap ClassLoader (null in Java API)
    ↑ loads rt.jar / java.base (JDK modules)
Extension ClassLoader
    ↑ loads ext directory (legacy)
Application / System ClassLoader
    ↑ loads classpath (your app)
Custom ClassLoaders
    ↑ OSGi, plugin systems, hot reload
```

**Delegation:** child asks parent first. Prevents core classes being replaced by malicious copies.

---

# 3. Custom ClassLoader Example

```java
public class PluginClassLoader extends ClassLoader {
    private final Path jarPath;

    public PluginClassLoader(Path jarPath, ClassLoader parent) {
        super(parent);
        this.jarPath = jarPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] bytes = loadClassBytes(name);
        return defineClass(name, bytes, 0, bytes.length);
    }

    private byte[] loadClassBytes(String name) throws ClassNotFoundException {
        String path = name.replace('.', '/') + ".class";
        try (InputStream in = Files.newInputStream(jarPath.resolve(path))) {
            return in.readAllBytes();
        } catch (IOException e) {
            throw new ClassNotFoundException(name, e);
        }
    }
}
```

---

# 4. JVM Runtime Data Areas

| Area | Thread-safe? | Stores | OOM type |
|------|--------------|--------|----------|
| **Heap** | Shared | Objects, arrays | `Java heap space` |
| **Metaspace** | Shared | Class metadata, static constants (post-Java 8) | `Metaspace` |
| **Java Stack** | Per-thread | Stack frames, local vars, operand stack | `StackOverflowError` |
| **Native Stack** | Per-thread | JNI native calls | native OOM |
| **PC Register** | Per-thread | Current instruction address | — |
| **Code Cache** | Shared | JIT compiled native code | — |

---

# 5. Heap Structure (Generational)

```text
┌─────────────────────────────────────┐
│  Young Generation                   │
│  ┌─────────┬──────────┬──────────┐  │
│  │  Eden   │ Survivor0│ Survivor1│  │
│  └─────────┴──────────┴──────────┘  │
├─────────────────────────────────────┤
│  Old Generation (Tenured)           │
└─────────────────────────────────────┘
```

New objects in Eden → minor GC survivors → S0/S1 → promoted to Old after threshold.

---

# 6. Stack Frame (Per Method Call)

```text
Frame contains:
  - Local variable table (params + locals)
  - Operand stack (bytecode operations)
  - Reference to constant pool
  - Return address
```

`StackOverflowError` — too deep recursion (exceeds `-Xss`).

---

# 7. PermGen vs Metaspace

| Java ≤7 | Java 8+ |
|---------|---------|
| PermGen (fixed size in heap) | Metaspace (native memory) |
| `PermGen space` OOM common | `-XX:MaxMetaspaceSize` to cap |

Class metadata moved out of heap → fewer PermGen OOMs; metaspace can grow until OS limit.

---

# 8. Important JVM Flags

```bash
-Xms512m          # initial heap
-Xmx2g            # max heap
-Xss1m            # thread stack size
-XX:MetaspaceSize=128m
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/dump.hprof
-verbose:class    # log class loading
```

---

# Interview Questions

## How many times is a class loaded?

Once per classloader namespace. Same class name in two classloaders = two different `Class` objects.

## Parent-first vs child-first?

Servlet containers (Tomcat) may use **inverted** delegation for web apps so app libs override container versions for specific packages.

## `Class.forName` vs `ClassLoader.loadClass`?

`forName` triggers initialization. `loadClass(name, false)` loads without initializing.

## Where are static variables stored?

Class metadata in Metaspace; **values** of static fields for primitives/refs live in Metaspace (Java 8+) as part of runtime constant/static storage linked to `Class` object.

## Difference heap vs stack for objects?

Object **reference** on stack (local var); object **instance** on heap. Escape analysis may scalar-replace on stack (JIT optimization).

---

# Quick Revision

```text
Load → Link → Init
Bootstrap → Extension → Application classloaders
Heap (objects), Metaspace (classes), Stack (frames)
Minor GC (young), Major/Full GC (old)
Delegation: parent loads first
```

---

*End of Day 30 JVM Classloader & Runtime Areas*
