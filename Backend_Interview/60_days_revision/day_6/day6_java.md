# Day 6 — Java

## Topics
- Comparable
- Comparator
- PriorityQueue
- Generics

---

# Comparable

Used for natural sorting.

```java
class Student implements Comparable<Student> {

    int marks;

    @Override
    public int compareTo(Student s) {
        return this.marks - s.marks;
    }
}
```

---

# Comparator

Custom sorting logic.

```java
Collections.sort(list,
        (a, b) -> a.name.compareTo(b.name));
```

---

# PriorityQueue

Heap implementation.

```java
PriorityQueue<Integer> pq =
        new PriorityQueue<>();
```

---

# Max Heap

```java
PriorityQueue<Integer> pq =
        new PriorityQueue<>(
                (a, b) -> b - a
        );
```

---

# Generics

Provide type safety.

```java
List<String> list =
        new ArrayList<>();
```

---

# Interview Questions

## Comparable vs Comparator?

| Comparable | Comparator |
|---|---|
| Natural sorting | Custom sorting |
| compareTo() | compare() |

---

# Quick Revision

```text
Comparable → natural sorting
Comparator → custom sorting
PriorityQueue → heap
```

---

*End of Day 6 Java*