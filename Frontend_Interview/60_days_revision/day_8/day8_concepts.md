# Day 8 — JavaScript & TypeScript Revision

**Topics:** map / filter / reduce (advanced) · Chaining · Composition · Performance pitfalls

---

## Table of Contents

1. [map — Advanced Patterns](#1-map--advanced-patterns)
2. [filter — Advanced Patterns](#2-filter--advanced-patterns)
3. [reduce — Advanced Patterns](#3-reduce--advanced-patterns)
4. [Chaining & Composition](#4-chaining--composition)
5. [Performance & Interview Traps](#5-performance--interview-traps)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. map — Advanced Patterns

### Foundation

`map` transforms each element → **new array of same length**. Does not mutate original.

```js
const nums = [1, 2, 3];
const doubled = nums.map((n) => n * 2); // [2, 4, 6]
```

### map with index & array

```js
["a", "b", "c"].map((val, index, arr) => ({
  val,
  index,
  isLast: index === arr.length - 1,
}));
// [{ val: "a", index: 0, isLast: false }, ...]
```

### Nested map (matrix transform)

```js
const matrix = [
  [1, 2],
  [3, 4],
];
const transposed = matrix[0].map((_, col) => matrix.map((row) => row[col]));
// [[1, 3], [2, 4]]
```

### map → object keys (Object.fromEntries)

```js
const users = [
  { id: 1, name: "A" },
  { id: 2, name: "B" },
];
const byId = Object.fromEntries(users.map((u) => [u.id, u]));
// { 1: { id: 1, name: "A" }, 2: { ... } }
```

### TypeScript — typed map

```ts
function mapWithIndex<T, U>(
  arr: T[],
  fn: (item: T, index: number) => U
): U[] {
  return arr.map(fn);
}

const ids = mapWithIndex(users, (u) => u.id); // number[]
```

### When NOT to use map

| Situation | Use instead |
|-----------|-------------|
| Side effects only (logging) | `forEach` |
| Filter + transform | `filter` then `map`, or single `reduce` |
| Early exit | `for...of` with `break` |
| Sparse arrays | `map` skips holes — use `flatMap` or loop |

---

## 2. filter — Advanced Patterns

### Foundation

Returns **subset** where predicate is truthy. Length ≤ original.

```js
const evens = [1, 2, 3, 4].filter((n) => n % 2 === 0); // [2, 4]
```

### Remove falsy / dedupe patterns

```js
const mixed = [0, "", "hello", null, undefined, 42];
mixed.filter(Boolean); // [ "hello", 42 ]

// Dedupe (simple types)
[...new Set(arr)]; // preferred for primitives
arr.filter((v, i, a) => a.indexOf(v) === i); // O(n²) — mention in interview
```

### filter + type narrowing (TypeScript)

```ts
function isDefined<T>(x: T | null | undefined): x is T {
  return x != null;
}

const maybeNames: (string | null)[] = ["a", null, "b"];
const names = maybeNames.filter(isDefined); // string[]
```

### Partition (filter twice vs reduce)

```js
function partition(arr, predicate) {
  return arr.reduce(
    (acc, item) => {
      acc[predicate(item) ? 0 : 1].push(item);
      return acc;
    },
    [[], []]
  );
}
const [pass, fail] = partition([1, 2, 3, 4], (n) => n % 2 === 0);
// pass: [2, 4], fail: [1, 3]
```

### Sliding-window connection (interview bridge)

Counting distinct elements in a window often uses **frequency map + filter logic**:

```js
// "At most K distinct" → expand/shrink window, track Map size
function atMostKDistinct(arr, k) {
  let left = 0,
    count = new Map(),
    total = 0;
  for (let right = 0; right < arr.length; right++) {
    count.set(arr[right], (count.get(arr[right]) ?? 0) + 1);
    while (count.size > k) {
      const l = arr[left++];
      count.set(l, count.get(l) - 1);
      if (count.get(l) === 0) count.delete(l);
    }
    total += right - left + 1; // all subarrays ending at right
  }
  return total;
}
```

---

## 3. reduce — Advanced Patterns

### Foundation

Fold array → **single value** (any type). Most flexible HOF.

```js
const sum = [1, 2, 3].reduce((acc, n) => acc + n, 0); // 6
```

### Group by (interview favorite)

```js
const orders = [
  { cat: "food", amt: 10 },
  { cat: "book", amt: 5 },
  { cat: "food", amt: 3 },
];

const grouped = orders.reduce((acc, o) => {
  (acc[o.cat] ??= []).push(o);
  return acc;
}, {});
// { food: [...], book: [...] }
```

### Count frequencies

```js
const freq = "aabbc".split("").reduce((m, ch) => {
  m[ch] = (m[ch] ?? 0) + 1;
  return m;
}, {});
// { a: 2, b: 2, c: 1 }
```

### pipe / compose with reduce

```js
const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((v, fn) => fn(v), x);

const clamp = (min, max) => (n) => Math.min(max, Math.max(min, n));
const double = (n) => n * 2;

pipe(clamp(0, 10), double)(7); // 14
```

### flatMap via reduce

```js
const nested = [[1, 2], [3], [4, 5]];
nested.reduce((acc, row) => acc.concat(row), []);
// or: nested.flatMap((row) => row)
```

### Implement map / filter with reduce

```js
const map = (arr, fn) => arr.reduce((acc, v, i) => (acc.push(fn(v, i)), acc), []);
const filter = (arr, pred) =>
  arr.reduce((acc, v) => (pred(v) ? acc.push(v) : null, acc), []);
```

### reduceRight

Same as reduce but right → left. Useful for nested function composition:

```js
const compose =
  (...fns) =>
  (x) =>
    fns.reduceRight((v, fn) => fn(v), x);
```

### Missing initial value trap

```js
[].reduce((a, b) => a + b); // TypeError — empty array, no initial
[].reduce((a, b) => a + b, 0); // 0 ✓
[1].reduce((a, b) => a + b); // 1 — uses first element as acc
```

---

## 4. Chaining & Composition

### Readable chain

```js
const result = users
  .filter((u) => u.active)
  .map((u) => ({ ...u, label: u.name.toUpperCase() }))
  .reduce((acc, u) => acc + u.score, 0);
```

### Multiple passes vs single reduce

| Approach | Pros | Cons |
|----------|------|------|
| chain map/filter | Readable, debuggable | Multiple array allocations |
| single reduce | One pass, fewer allocations | Harder to read |

Interview answer: prefer clarity; optimize with one reduce only when profiling shows bottleneck.

### Immutable update pattern (React state)

```js
setItems((prev) =>
  prev
    .filter((item) => item.id !== deletedId)
    .map((item) => (item.id === id ? { ...item, done: true } : item))
);
```

---

## 5. Performance & Interview Traps

| Trap | Detail |
|------|--------|
| `map` then `filter` | Two O(n) passes — often fine |
| `filter` inside `map` | O(n²) if nested on same array |
| Mutating `acc` in reduce | OK if intentional; return acc always |
| `async` in map | Returns array of Promises — need `Promise.all` |
| Sparse arrays | `map`/`filter`/`reduce` skip empty slots |

### async map

```js
const urls = ["/a", "/b"];
const data = await Promise.all(urls.map((url) => fetch(url).then((r) => r.json())));
```

### Lazy alternatives (mention)

- Generators for large datasets
- `Array.prototype.flatMap` for map+flatten in one step

---

## 6. Interview Quick Index

| Question | Section |
|----------|---------|
| map vs forEach | [§1](#1-map--advanced-patterns) |
| Implement map with reduce | [§3](#3-reduce--advanced-patterns) |
| Group by with reduce | [§3](#3-reduce--advanced-patterns) |
| filter for type narrowing | [§2](#2-filter--advanced-patterns) |
| Empty reduce without initial | [§3 — trap](#3-reduce--advanced-patterns) |
| pipe / compose | [§3–§4](#4-chaining--composition) |
| Link to sliding window | [§2 — atMostKDistinct](#2-filter--advanced-patterns) |

---

## Day 8 Cheat Sheet

```
map     → transform each → same length
filter  → keep if truthy → shorter/equal
reduce  → fold to any value; groupBy, freq, pipe
chain   → readable; single reduce when perf matters
async   → map + Promise.all, not await inside bare map loop
```

---

*End of Day 8 revision*
