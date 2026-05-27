# Day 3 — Java Collections Framework

**Topics:** ArrayList · LinkedList · HashMap · Internal Working · Interview Questions

---

# 1. Java Collections Framework

Java Collections Framework provides:
- Dynamic data structures
- Utility algorithms
- Interfaces and implementations

---

# Collection Hierarchy

```text
Iterable
   ↓
Collection
   ├── List
   │     ├── ArrayList
   │     └── LinkedList
   │
   └── Set

Map
   └── HashMap
```

---

# 2. ArrayList

## What is ArrayList?

Dynamic array implementation in Java.

```java
ArrayList<Integer> list = new ArrayList<>();
```

---

## Features

- Dynamic resizing
- Maintains insertion order
- Fast random access

---

## Internal Working

Internally uses:
```text
Resizable Array
```

Default capacity:
```text
10
```

When full:
```text
newCapacity = oldCapacity + oldCapacity/2
```

---

## Common Methods

```java
list.add(10);
list.get(0);
list.remove(0);
list.contains(10);
```

---

## Time Complexity

| Operation | Complexity |
|---|---|
| Add | O(1) |
| Get | O(1) |
| Remove Middle | O(n) |

---

# 3. LinkedList

## What is LinkedList?

Doubly linked list implementation.

```java
LinkedList<Integer> list = new LinkedList<>();
```

---

## Internal Structure

Each node contains:
- previous pointer
- data
- next pointer

---

## Advantages

- Fast insertion/deletion
- No shifting required

---

## Time Complexity

| Operation | Complexity |
|---|---|
| Add First | O(1) |
| Remove First | O(1) |
| Search | O(n) |

---

# 4. HashMap

## What is HashMap?

Stores:
```text
key → value
```

pairs.

---

## Example

```java
HashMap<Integer, String> map = new HashMap<>();

map.put(1, "Java");
map.put(2, "Spring");

System.out.println(map.get(1));
```

---

## Internal Working

Uses:
- Hashing
- Buckets
- Linked List
- Red Black Tree (after threshold)

---

## Collision

When two keys map to same bucket.

Handled using:
- Linked List
- Treeification

---

## Time Complexity

| Operation | Complexity |
|---|---|
| Put | O(1) |
| Get | O(1) |
| Remove | O(1) |

Worst Case:
```text
O(n)
```

---

# HashMap Internal Flow

```text
key.hashCode()
      ↓
hashing
      ↓
bucket index
      ↓
store Entry Node
```

---

# ArrayList vs LinkedList

| Feature | ArrayList | LinkedList |
|---|---|---|
| Access | Fast | Slow |
| Insert/Delete | Slow | Fast |
| Internal Structure | Dynamic Array | Nodes |

---

# Interview Questions

## Difference between ArrayList and LinkedList?
ArrayList is faster for searching.
LinkedList is better for insertion/deletion.

---

## Why HashMap is fast?
Because hashing gives near constant-time lookup.

---

## Is HashMap thread safe?
No.

Use:
```java
ConcurrentHashMap
```

for multithreading.

---

## Difference between HashMap and Hashtable?

| HashMap | Hashtable |
|---|---|
| Not synchronized | Synchronized |
| Faster | Slower |
| Allows null | No null allowed |

---

# Quick Revision

```text
ArrayList → Dynamic Array
LinkedList → Nodes with pointers
HashMap → Key-value hashing structure
```

---

*End of Day 3 Collections*