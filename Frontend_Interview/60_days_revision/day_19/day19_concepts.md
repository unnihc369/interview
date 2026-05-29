# Day 19 — Set & Map Performance

**Week 3 — Two Pointers** · **Topics:** `Set` · `Map` · Big-O lookups · Key equality · WeakMap/WeakSet

---

## Table of Contents

1. [Set Fundamentals](#1-set-fundamentals)
2. [Map Fundamentals](#2-map-fundamentals)
3. [Performance Comparison](#3-performance-comparison)
4. [Object vs Map vs Set](#4-object-vs-map-vs-set)
5. [Two-Pointer + Set Patterns](#5-two-pointer--set-patterns)
6. [WeakMap & WeakSet](#6-weakmap--weakset)
7. [React & Frontend Patterns](#7-react--frontend-patterns)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. Set Fundamentals

### Creation & basic ops

```js
const s = new Set([1, 2, 2, 3]); // {1, 2, 3}
s.add(4);
s.has(2);    // true
s.delete(1);
s.size;      // 3
s.clear();
```

### Dedup array

```js
const unique = [...new Set(arr)];
// or
Array.from(new Set(arr));
```

### Set operations (manual)

```js
function union(a, b) {
  return new Set([...a, ...b]);
}

function intersection(a, b) {
  return new Set([...a].filter((x) => b.has(x)));
}

function difference(a, b) {
  return new Set([...a].filter((x) => !b.has(x)));
}
```

### Iteration order

Insertion order (like Map keys).

---

## 2. Map Fundamentals

### Key-value with any key type

```js
const m = new Map();
m.set("name", "Ada");
m.set(42, "answer");
m.set({ id: 1 }, "obj key"); // object identity as key

m.get("name"); // "Ada"
m.has(42);       // true
m.delete(42);
```

### vs plain object

| Feature | Map | Object |
|---------|-----|--------|
| Key types | Any | string / symbol |
| Size | `.size` | manual count |
| Iteration order | Insertion | mostly insertion (integer keys reorder) |
| Default keys | none | prototype chain |

### Frequent use: index by id

```js
const byId = new Map(users.map((u) => [u.id, u]));
const user = byId.get(id);
```

### Map from entries

```js
new Map([["a", 1], ["b", 2]]);
Object.entries(obj); // → Map
```

---

## 3. Performance Comparison

### Average-case Big-O (V8 / modern engines)

| Operation | Set | Map | Array.indexOf | Object key |
|-----------|-----|-----|---------------|------------|
| has / get | O(1) | O(1) | O(n) | O(1)* |
| add / set | O(1) | O(1) | — | O(1) |
| delete | O(1) | O(1) | — | O(1) |
| iterate all | O(n) | O(n) | O(n) | O(n) |

*Object: only string/symbol keys; no hash for arbitrary objects as keys.

### When Set beats array

```js
// O(n²) — bad for large n
function hasDuplicate(arr) {
  for (let i = 0; i < arr.length; i++)
    for (let j = i + 1; j < arr.length; j++)
      if (arr[i] === arr[j]) return true;
  return false;
}

// O(n)
function hasDuplicateSet(arr) {
  const seen = new Set();
  for (const x of arr) {
    if (seen.has(x)) return true;
    seen.add(x);
  }
  return false;
}
```

### Map for two-sum complement

```js
function twoSum(nums, target) {
  const seen = new Map(); // value → index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(nums[i], i);
  }
  return [];
}
```

---

## 4. Object vs Map vs Set

### Choose Set when

- Unique membership tests
- Dedup
- Visited tracking in graphs / cycles (LC 457)

### Choose Map when

- Keys are not strings
- Frequent add/delete/size checks
- Preserve insertion order with mixed key types

### Choose Object when

- JSON-serializable record
- Fixed schema (DTO)
- Prototype / methods on record

---

## 5. Two-Pointer + Set Patterns

### Intersection of two arrays (LC 349)

```js
function intersection(nums1, nums2) {
  const set2 = new Set(nums2);
  const res = new Set();
  for (const n of nums1) {
    if (set2.has(n)) res.add(n);
  }
  return [...res];
}
```

### Circular array loop detection (LC 457)

Use Set of visited **indices** while simulating movement with fast/slow or visited set.

```js
function circularArrayLoop(nums) {
  const n = nums.length;
  for (let start = 0; start < n; start++) {
    const seen = new Set();
    let i = start;
    const forward = nums[start] > 0;
    while (true) {
      if (seen.has(i)) {
        if (seen.size > 1) return true;
        break;
      }
      seen.add(i);
      const next = (i + nums[i] % n + n) % n;
      if (next === i) break;
      if ((nums[next] > 0) !== forward) break;
      i = next;
    }
  }
  return false;
}
```

---

## 6. WeakMap & WeakSet

### WeakMap

- Keys must be **objects**
- Keys are weakly held — GC when no other refs
- Not iterable; no `.size`

```js
const cache = new WeakMap();
function process(domNode) {
  if (cache.has(domNode)) return cache.get(domNode);
  const result = expensive(domNode);
  cache.set(domNode, result);
  return result;
}
```

### WeakSet

Track object membership without preventing GC (e.g. "already visited" DOM nodes).

---

## 7. React & Frontend Patterns

### Selected rows Set

```tsx
const [selected, setSelected] = useState<Set<string>>(new Set());

const toggle = (id: string) =>
  setSelected((prev) => {
    const next = new Set(prev);
    if (next.has(id)) next.delete(id);
    else next.add(id);
    return next;
  });
```

### Memoization cache Map

```js
const cache = new Map();
function fetchUser(id) {
  if (cache.has(id)) return Promise.resolve(cache.get(id));
  return api.get(id).then((u) => { cache.set(id, u); return u; });
}
```

### List diff (Day 19 MC)

```js
function diffKeys(oldList, newList) {
  const oldSet = new Set(oldList.map((x) => x.id));
  const newSet = new Set(newList.map((x) => x.id));
  return {
    added: newList.filter((x) => !oldSet.has(x.id)),
    removed: oldList.filter((x) => !newSet.has(x.id)),
    kept: newList.filter((x) => oldSet.has(x.id)),
  };
}
```

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| Set vs array for lookup | [§3](#3-performance-comparison) |
| Map vs Object | [§4](#4-object-vs-map-vs-set) |
| Dedup one-liner | [§1](#1-set-fundamentals) |
| WeakMap use case | [§6](#6-weakmap--weakset) |
| Intersection pattern | [§5](#5-two-pointer--set-patterns) |

---

## Day 19 Cheat Sheet

```
Set     → unique values; .has O(1); dedup [...new Set(a)]
Map     → any key; .get/.set O(1); use for index by id
Object  → JSON records; string keys
WeakMap → object keys, GC-friendly, no iterate
Array lookup → O(n); Set/Map → O(1) average
```

---

*End of Day 19 concepts*
