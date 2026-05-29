# Day 17 — Java Reflection API

**Topics:** Class Inspection · Dynamic Method Invocation · Annotations at Runtime · Interview Questions

---

# 1. What is Reflection?

Reflection allows Java program to inspect and modify classes, methods, fields at **runtime** — even private members.

Package: `java.lang.reflect`

---

# Why Reflection Matters in Backend

- Spring IoC container creates beans dynamically
- JPA/Hibernate maps entities to tables
- Serialization frameworks
- Testing frameworks (JUnit, Mockito)
- Custom annotation processing

---

# 2. Getting Class Object

```java
// Method 1: .class literal
Class<?> clazz = String.class;

// Method 2: getClass() on instance
String s = "hello";
Class<?> clazz2 = s.getClass();

// Method 3: Class.forName (dynamic loading)
Class<?> clazz3 = Class.forName("com.example.User");
```

---

# 3. Inspecting Class Metadata

```java
Class<?> clazz = User.class;

System.out.println(clazz.getName());           // fully qualified name
System.out.println(clazz.getSimpleName());     // User
System.out.println(clazz.getSuperclass());     // parent class
System.out.println(clazz.getInterfaces());     // implemented interfaces
System.out.println(clazz.getModifiers());      // public, abstract, etc.
System.out.println(clazz.isAnnotationPresent(Entity.class));
```

---

# 4. Accessing Fields

```java
Class<?> clazz = User.class;
Field field = clazz.getDeclaredField("email");
field.setAccessible(true);  // bypass private access

User user = new User();
field.set(user, "test@example.com");
String value = (String) field.get(user);
```

---

# Iterate All Fields

```java
for (Field field : clazz.getDeclaredFields()) {
    System.out.println(field.getName() + " : " + field.getType());
}
```

---

# 5. Accessing Methods

```java
Method method = clazz.getDeclaredMethod("setName", String.class);
method.setAccessible(true);
method.invoke(user, "Alice");

// No-arg method
Method getName = clazz.getMethod("getName");
String name = (String) getName.invoke(user);
```

---

# 6. Creating Objects Dynamically

```java
Class<?> clazz = Class.forName("com.example.User");
Constructor<?> constructor = clazz.getDeclaredConstructor();
User user = (User) constructor.newInstance();

// With parameters
Constructor<?> paramCtor = clazz.getDeclaredConstructor(String.class, String.class);
User user2 = (User) paramCtor.newInstance("Bob", "bob@example.com");
```

---

# 7. Reading Annotations via Reflection

```java
Method method = clazz.getMethod("processOrder");

if (method.isAnnotationPresent(Transactional.class)) {
    Transactional tx = method.getAnnotation(Transactional.class);
    System.out.println("Propagation: " + tx.propagation());
    System.out.println("ReadOnly: " + tx.readOnly());
}
```

Requires `@Retention(RetentionPolicy.RUNTIME)`.

---

# 8. Practical Example — Simple DI Container

```java
public class SimpleContainer {
    private final Map<Class<?>, Object> beans = new HashMap<>();

    public <T> T getBean(Class<T> type) {
        return type.cast(beans.computeIfAbsent(type, this::createInstance));
    }

    private Object createInstance(Class<?> type) {
        try {
            Constructor<?> ctor = type.getDeclaredConstructor();
            ctor.setAccessible(true);
            Object instance = ctor.newInstance();

            for (Field field : type.getDeclaredFields()) {
                if (field.isAnnotationPresent(Inject.class)) {
                    field.setAccessible(true);
                    field.set(instance, getBean(field.getType()));
                }
            }
            return instance;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

This mirrors how Spring resolves `@Autowired` dependencies.

---

# 9. Method Handles vs Reflection (Java 7+)

| Reflection | MethodHandle |
|------------|--------------|
| Slower, checked exceptions | Faster, type-safe |
| Easy to use | More verbose setup |
| Legacy APIs | Preferred for performance-critical code |

---

# 10. Security & Performance Concerns

## Performance

Reflection is slower than direct calls — JVM cannot inline as aggressively.

## Security

`setAccessible(true)` bypasses encapsulation. Java 9+ module system restricts deep reflection.

## Best Practice

Use reflection in frameworks, not business logic. Prefer interfaces and dependency injection.

---

# Interview Questions

## How does Spring create beans using reflection?

1. Component scan finds `@Component` classes
2. Reads constructor parameters via reflection
3. Resolves dependencies recursively
4. Calls `Constructor.newInstance()` to create bean
5. Applies `@PostConstruct`, `@Autowired` field injection via reflection

---

## Can reflection access private methods?

Yes, with `setAccessible(true)`. Breaks encapsulation — use only in frameworks/testing.

---

## Difference between getMethod and getDeclaredMethod?

| getMethod | getDeclaredMethod |
|-----------|-------------------|
| Public methods only | All methods (including private) |
| Includes inherited | Declared in class only |

---

## What is Class.forName used for?

Dynamic class loading — JDBC drivers traditionally loaded this way:

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

---

## Reflection vs Compile-time annotation processing?

| Reflection (Runtime) | Annotation Processor (Compile-time) |
|---------------------|-------------------------------------|
| Flexible, slower | Fast, generates code at compile time |
| Spring, Hibernate | Lombok, MapStruct |

---

# Quick Revision

```text
Class.forName / .class → get Class object
getDeclaredField/Method → inspect members
setAccessible(true)    → access private members
invoke/newInstance     → dynamic call/create
isAnnotationPresent    → runtime annotation check
Spring IoC             → reflection-based DI container
```

---

*End of Day 17 Reflection API*
