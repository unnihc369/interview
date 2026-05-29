# Day 36 — Java Bytecode & ASM Introduction

**Topics:** JVM Bytecode · Class File Structure · ASM Library · Interview Questions

---

# 1. What is Java Bytecode?

Bytecode = platform-independent instruction set executed by the JVM.

```text
.java source  →  javac  →  .class bytecode  →  JVM (interpreter + JIT)  →  native machine code
```

Interview angle: bytecode is **not** machine code — JVM interprets or compiles it per platform.

---

# 2. Class File Structure

| Section | Purpose |
|---------|---------|
| Magic `0xCAFEBABE` | Identifies valid class file |
| Version | Major/minor Java version |
| Constant Pool | Literals, class/method refs |
| Access Flags | public, final, interface, etc. |
| This/Super Class | Inheritance info |
| Fields | Field descriptors |
| Methods | Code attribute with bytecode |
| Attributes | Line numbers, stack map, etc. |

---

# 3. Reading Bytecode with javap

```bash
javac Hello.java
javap -c -v Hello.class
```

Sample output:

```text
0: aload_0
1: invokespecial #1  // Method java/lang/Object."<init>":()V
4: return
```

Common opcodes:

| Opcode | Meaning |
|--------|---------|
| `aload_0` | Load `this` reference |
| `iload_1` | Load int local var 1 |
| `invokevirtual` | Instance method call |
| `invokestatic` | Static method call |
| `getfield` / `putfield` | Field access |
| `ifeq` | Branch if int == 0 |

---

# 4. Operand Stack & Local Variables

Each method has:

- **Local variable table** — parameters + locals (`this`, args)
- **Operand stack** — intermediate values for operations

```java
int add(int a, int b) {
    return a + b;
}
```

Rough bytecode flow: load `a`, load `b`, `iadd`, `ireturn`.

---

# 5. Why ASM?

ASM = lightweight bytecode manipulation library (faster than BCEL, lower-level than Javassist).

Use cases:

- AOP frameworks (AspectJ weaving)
- Mockito / bytecode proxies
- Code coverage tools (JaCoCo)
- Performance agents
- Custom annotations processors at runtime

---

# 6. ASM Core Concepts

| Concept | Role |
|---------|------|
| `ClassReader` | Parse existing `.class` |
| `ClassWriter` | Generate new `.class` |
| `ClassVisitor` | Transform during visit |
| `MethodVisitor` | Inject/modify method bytecode |
| `AdviceAdapter` | Easier method entry/exit hooks |

---

# 7. Simple ASM Example — Add Timing Log

```java
import org.objectweb.asm.*;

public class TimingClassVisitor extends ClassVisitor {
    public TimingClassVisitor(ClassVisitor cv) {
        super(Opcodes.ASM9, cv);
    }

    @Override
    public MethodVisitor visitMethod(int access, String name, String descriptor,
                                     String signature, String[] exceptions) {
        MethodVisitor mv = super.visitMethod(access, name, descriptor, signature, exceptions);
        if ("<init>".equals(name) || (access & Opcodes.ACC_ABSTRACT) != 0) {
            return mv;
        }
        return new AdviceAdapter(Opcodes.ASM9, mv, access, name, descriptor) {
            @Override
            protected void onMethodEnter() {
                mv.visitMethodInsn(INVOKESTATIC, "java/lang/System", "nanoTime", "()J", false);
                mv.visitVarInsn(LSTORE, 1);
            }
            @Override
            protected void onMethodExit(int opcode) {
                mv.visitMethodInsn(INVOKESTATIC, "java/lang/System", "nanoTime", "()J", false);
                mv.visitVarInsn(LLOAD, 1);
                mv.visitInsn(LSUB);
                // log elapsed — simplified
            }
        };
    }
}
```

---

# 8. ASM vs Reflection vs Proxy

| Approach | When |
|----------|------|
| Reflection | Inspect/call at runtime; slower |
| JDK Dynamic Proxy | Interface-only proxies |
| CGLIB / ByteBuddy / ASM | Subclass-based proxies, bytecode weaving |

Spring AOP uses proxies (JDK or CGLIB); Hibernate, Mockito use bytecode libs heavily.

---

# 9. Bytecode Verification

JVM verifies before execution:

- Stack depth consistent
- Type safety on stack
- No illegal field/method access

Security: prevents malicious bytecode from corrupting JVM state.

---

# 10. Advanced Interview Topics

## invokedynamic (Java 7+)

Bootstrap methods for lambda, method references, dynamic languages on JVM.

```java
Runnable r = () -> System.out.println("hi");
// desugared to invokedynamic + LambdaMetafactory
```

## StackMapTable

Post-Java 6 verification uses stack maps instead of full dataflow analysis — smaller class files, faster verification.

---

# Interview Questions

## Q1. Difference between bytecode and machine code?

Bytecode is JVM-specific, portable. Machine code is CPU-specific (x86, ARM).

## Q2. Why learn ASM as backend engineer?

Understand how frameworks instrument code, debug proxy issues, build agents, optimize hot paths with JIT hints.

## Q3. How does Mockito mock final classes?

Bytecode subclassing (inline mock maker) or agent attaching — modifies classes at load time via instrumentation.

## Q4. What is `invokespecial` vs `invokevirtual`?

`invokespecial` — constructors, private methods, super calls. `invokevirtual` — virtual dispatch on instance methods.

## Q5. Is modifying bytecode safe in production?

Only with rigorous testing; classloader isolation; prefer official APIs (Instrumentation API) over ad-hoc edits.

---

# One-Line Revision

```text
Bytecode = JVM instructions; ASM = visit/transform .class for AOP, mocks, agents.
javap -c shows opcodes; max freq scheduling unrelated — use heap/greedy for string problems.
```

---

*End of Day 36 Java*
