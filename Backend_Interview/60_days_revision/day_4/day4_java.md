# Day 4 - Java Generics & Comparison APIs

---

# 1. Generics

---

# Definition

Generics allow writing reusable, type-safe code.

```text
Write once, use with different data types.
```

Without generics:

```java
List list = new ArrayList();
list.add("abc");
Integer n = (Integer) list.get(0); // Runtime ClassCastException
```

With generics:

```java
List<String> list = new ArrayList<>();
list.add("abc");
String s = list.get(0); // Type-safe
```

---

# Why Generics?

- Compile-time type safety
- No explicit type casting
- Reusable API design
- Better readability

---

# Generic Class Example

```java
class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

Box<Integer> intBox = new Box<>();
intBox.set(10);
Integer x = intBox.get();
```

---

# Generic Method Example

```java
class Util {
    public static <T> void printArray(T[] arr) {
        for (T item : arr) {
            System.out.println(item);
        }
    }
}
```

---

# Bounded Generics

```java
public static <T extends Number> double sum(List<T> nums) {
    double total = 0;
    for (T n : nums) total += n.doubleValue();
    return total;
}
```

`T extends Number` means only numeric types allowed.

---

# Wildcards

| Syntax | Meaning |
|--------|---------|
| `<?>` | Unknown type |
| `<? extends T>` | Upper bounded (read-only style) |
| `<? super T>` | Lower bounded (write-friendly style) |

```java
List<? extends Number> nums = List.of(1, 2, 3);
Number n = nums.get(0);   // read OK
// nums.add(4);           // not allowed
```

---

# Type Erasure (Interview Important)

Java removes generic type info at runtime.

```text
List<String> and List<Integer> become List at runtime
```

Because of erasure:

- Cannot create `new T()`
- Cannot use `instanceof List<String>`

---

# 2. Comparable vs Comparator

---

# Comparable

Defines **natural ordering** inside the class itself.

```java
public class Employee implements Comparable<Employee> {
    int id;
    String name;

    @Override
    public int compareTo(Employee other) {
        return this.id - other.id; // natural order by id
    }
}
```

Usage:

```java
Collections.sort(employeeList); // uses compareTo
```

---

# Comparator

Defines **custom ordering** outside the class.

```java
Comparator<Employee> byName = (a, b) -> a.name.compareTo(b.name);
Comparator<Employee> byIdDesc = (a, b) -> b.id - a.id;

employeeList.sort(byName);
employeeList.sort(byIdDesc);
```

---

# Comparable vs Comparator

| Comparable | Comparator |
|------------|------------|
| Inside class | Separate class/lambda |
| Single default sort | Multiple custom sorts |
| `compareTo()` | `compare()` |
| Modifies domain class | No need to modify domain class |

---

# Java 8 Comparator Helpers

```java
employeeList.sort(Comparator.comparing(Employee::getName));
employeeList.sort(Comparator.comparingInt(Employee::getId));
employeeList.sort(Comparator.comparing(Employee::getName).reversed());
employeeList.sort(
    Comparator.comparing(Employee::getDept)
              .thenComparing(Employee::getName)
);
```

---

# Practical Interview Example

Sort `Student` by:

1. marks descending
2. name ascending

```java
students.sort(
    Comparator.comparingInt(Student::getMarks).reversed()
              .thenComparing(Student::getName)
);
```

---

# Common Interview Questions

## Q1. When to use Comparable?

When object has one natural ordering.

Example:

- `String` alphabetical
- `Integer` numeric

---

## Q2. When to use Comparator?

When multiple sorting logic is needed.

Example:

- sort users by age
- sort users by salary
- sort users by name

---

## Q3. Can we use both?

Yes.

- `Comparable` for default sort
- `Comparator` for runtime custom sort

---

# One-Line Revision

```text
Generics = compile-time type safety and reusable APIs.
Comparable = default sorting inside class.
Comparator = custom sorting outside class.
```

---

*End of Day 4 Java*
