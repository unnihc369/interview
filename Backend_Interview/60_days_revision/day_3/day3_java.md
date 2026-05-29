# Day 3 — Java Collections Framework

**Topics:** String · equals/hashCode · ArrayList · LinkedList · HashMap · LinkedHashMap · TreeMap · Fail-fast · Interview Questions

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
   ├── HashMap
   ├── LinkedHashMap
   └── TreeMap
```

---

# 2. String in Java (Interview Critical)

## Immutability

`String` objects are **immutable** — once created, content cannot change.

```java
String s = "Hello";
s = s + " World"; // creates NEW String object; original unchanged
```

Why immutable?

- **String pool** reuse (memory efficiency)
- **Thread-safe** without synchronization
- Safe as **HashMap keys** (hashCode never changes)

---

## String Pool & `intern()`

```java
String a = "Java";           // pool
String b = "Java";           // same pool reference
String c = new String("Java"); // heap object

System.out.println(a == b);      // true (same pool ref)
System.out.println(a == c);      // false (different objects)
System.out.println(a.equals(c)); // true (content equal)

String d = c.intern(); // moves/gets pool reference
System.out.println(a == d); // true
```

---

## StringBuilder vs StringBuffer

| | StringBuilder | StringBuffer |
|---|---------------|--------------|
| Thread-safe | No | Yes (synchronized) |
| Performance | Faster | Slower |
| Use when | Single thread concat | Legacy multi-thread (rare) |

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) sb.append(i);
String result = sb.toString();
```

**Interview:** Never use `+` in loops for strings — O(n²) due to new objects each concat.

---

# 3. equals() and hashCode() Contract

If two objects are **equal** (`equals` returns true), they **must** have the **same hashCode**.

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User user)) return false;
    return Objects.equals(id, user.id);
}

@Override
public int hashCode() {
    return Objects.hash(id); // must use same fields as equals
}
```

## Why it matters for HashMap

1. `hashCode()` → bucket index  
2. `equals()` → resolve collisions in bucket  

Break the contract → HashMap/HashSet lookups fail silently.

## Rules

| Rule | Detail |
|------|--------|
| Reflexive | `a.equals(a)` always true |
| Symmetric | `a.equals(b)` ↔ `b.equals(a)` |
| Transitive | if a=b and b=c then a=c |
| Consistent | same result if state unchanged |
| null | `a.equals(null)` → false |

**Java records** auto-generate correct `equals`/`hashCode`.

---

# 4. ArrayList

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

# 5. LinkedList

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

# 6. HashMap

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

## Load Factor & Treeify (Java 8+)

- Default **load factor = 0.75** — rehash when 75% full
- Default initial capacity **16** buckets
- When bucket has **> 8** nodes → converts linked list to **Red-Black Tree** (O(log n) worst case)
- Treeify only if map size ≥ 64

---

# 7. LinkedHashMap

Maintains **insertion order** (or access order if `accessOrder=true`).

```java
LinkedHashMap<String, Integer> map = new LinkedHashMap<>();
map.put("a", 1);
map.put("b", 2);
// iteration order: a, b
```

**Use case:** LRU cache — override `removeEldestEntry()` to evict oldest when size > capacity.

```java
LinkedHashMap<K, V> lru = new LinkedHashMap<>(16, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > MAX_CAPACITY;
    }
};
```

---

# 8. TreeMap

**Red-Black tree** — keys always **sorted**.

```java
TreeMap<String, Integer> map = new TreeMap<>();
map.put("banana", 2);
map.put("apple", 1);
// iteration: apple, banana

map.floorKey("banana");  // largest key ≤ banana
map.ceilingKey("cherry"); // smallest key ≥ cherry
```

| Operation | Complexity |
|-----------|------------|
| put/get/remove | O(log n) |
| firstKey/lastKey | O(log n) |

Requires keys implement `Comparable` or pass `Comparator`.

---

# 9. PriorityQueue (Min-Heap)

Binary heap — smallest element at head (default min-heap).

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();       // min-heap
PriorityQueue<Integer> maxPq = new PriorityQueue<>(Collections.reverseOrder());

pq.offer(5);
pq.offer(1);
pq.poll(); // 1 (smallest)
```

| Operation | Complexity |
|-----------|------------|
| offer/poll | O(log n) |
| peek | O(1) |

**Not thread-safe** — use `PriorityBlockingQueue` for concurrency.

---

# 10. Fail-Fast vs Fail-Safe Iterators

## Fail-Fast (ArrayList, HashMap)

Throws `ConcurrentModificationException` if collection modified during iteration (except via iterator's own `remove()`).

```java
List<String> list = new ArrayList<>(List.of("a", "b"));
for (String s : list) {
    list.remove(s); // ConcurrentModificationException
}
```

Uses **modCount** — detects structural change during iteration.

## Fail-Safe (ConcurrentHashMap, CopyOnWriteArrayList)

Iterates over **snapshot/copy** — no exception, may not see latest writes.

| | Fail-Fast | Fail-Safe |
|---|-----------|-----------|
| Exception | Yes | No |
| Memory | Lower | Higher (copy) |
| Examples | ArrayList, HashMap | ConcurrentHashMap, COWAL |

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

## HashMap vs LinkedHashMap vs TreeMap?

| | HashMap | LinkedHashMap | TreeMap |
|---|---------|---------------|---------|
| Order | None | Insertion/access | Sorted keys |
| null key | 1 allowed | 1 allowed | Not allowed |
| Performance | O(1) avg | O(1) avg | O(log n) |

---

## Why break equals/hashCode contract breaks HashMap?

Lookup uses hashCode for bucket, equals for collision chain. Unequal hashCodes for equal objects → entry stored but never found.

---

## Fail-fast vs fail-safe?

Fail-fast throws on concurrent modification during iteration. Fail-safe iterates snapshot — no exception, stale data possible.

---

# Quick Revision

```text
String → immutable, pool, use StringBuilder in loops
equals/hashCode → contract required for HashMap keys
ArrayList → Dynamic Array
LinkedList → Nodes with pointers
HashMap → hashing, load factor 0.75, treeify at 8
LinkedHashMap → insertion order / LRU
TreeMap → sorted keys, Red-Black tree
PriorityQueue → min-heap O(log n)
Fail-fast → ConcurrentModificationException
```

---

*End of Day 3 Collections*