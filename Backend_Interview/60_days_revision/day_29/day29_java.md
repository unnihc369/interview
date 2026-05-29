# Day 29 — Java Modules (JPMS)

**Topics:** Java Platform Module System · module-info.java · Encapsulation · Interview Questions

---

# 1. Why Modules? (Java 9+)

Before JPMS, JAR classpath was flat — any public class accessible. Problems:

- **Encapsulation leaks** — internal APIs used accidentally (e.g., `sun.misc.Unsafe`)
- **Classpath hell** — version conflicts, fat JARs
- **No compile-time boundary** between libraries

JPMS introduces **strong encapsulation** at compile and runtime.

---

# 2. Key Concepts

| Term | Meaning |
|------|---------|
| Module | Named set of packages with explicit exports |
| `module-info.java` | Module descriptor at module root |
| `requires` | Dependency on another module |
| `exports` | Makes package public to other modules |
| `opens` | Allows reflection into package (JPA, Jackson) |
| `provides` / `uses` | Service Provider Interface (SPI) |

---

# 3. module-info.java Example

```java
module com.example.orders {
    requires java.sql;
    requires com.example.common;
    requires transitive spring.context; // re-export to consumers

    exports com.example.orders.api;
    exports com.example.orders.dto to com.example.web;

    opens com.example.orders.entity to org.hibernate.orm.core;

    provides com.example.orders.PaymentGateway
        with com.example.orders.StripePaymentGateway;
}
```

---

# 4. Module Types

```text
Named application module  → has module-info.java
Automatic module          → JAR without module-info on module path (name from JAR)
Unnamed module            → classpath JARs (legacy, no strong encapsulation)
Platform modules          → java.base, java.sql, etc. (JDK itself is modular)
```

---

# 5. java.base — The Foundation

Every module implicitly requires `java.base`. It exports core packages (`java.lang`, `java.util`, etc.).

JDK internal packages (`jdk.internal.*`) are **not exported** — strong encapsulation by default.

---

# 6. Migration Strategies

```text
Classpath only (legacy)     → all code in unnamed module
Module path (strict)        → full JPMS enforcement
Mixed (transitional)        → app modular, some deps automatic modules
```

**Tips for interviews:**

- Start modularizing leaf libraries first
- Use `jdeps` to analyze dependencies
- `--add-opens` / `--add-exports` as JVM flags for legacy libs (temporary)

---

# 7. CLI Commands

```bash
# Compile module
javac -d out --module-source-path src $(find src -name "*.java")

# Run module
java --module-path out --module com.example.orders/com.example.orders.Main

# List module contents
java --list-modules
jar --describe-module --file myapp.jar
```

---

# 8. SPI with Modules

```java
// Provider module
module com.payment.stripe {
    requires com.payment.api;
    provides com.payment.api.PaymentService
        with com.payment.stripe.StripePaymentService;
}

// Consumer module
module com.checkout {
    requires com.payment.api;
    uses com.payment.api.PaymentService;
}

// ServiceLoader still works; module system wires providers
```

---

# Interview Questions

## JPMS vs OSGi?

Both modularize JVM apps. JPMS is built into JDK 9+; OSGi is older, dynamic lifecycle, heavier. Spring Boot apps often stay on classpath; JPMS used when strong boundaries needed.

## `exports` vs `opens`?

`exports` — compile-time and runtime access to types. `opens` — allows deep reflection (frameworks scanning entities). Can open to specific modules only.

## `requires transitive`?

If module A `requires transitive B`, any module requiring A automatically reads B. Used for API modules that expose types from dependencies.

## Can two modules export same package?

No — split packages forbidden on module path. Merge or refactor.

## Does Spring Boot require modules?

No. Most Boot apps use classpath. Modular Boot possible but not default; `--add-opens` common for reflection.

---

# Quick Revision

```text
module-info.java → requires, exports, opens, provides/uses
java.base        → implicit dependency
Automatic module → legacy JAR on module path
Unnamed module    → classpath (no encapsulation)
jdeps            → dependency analysis tool
```

---

*End of Day 29 Java Modules (JPMS)*
