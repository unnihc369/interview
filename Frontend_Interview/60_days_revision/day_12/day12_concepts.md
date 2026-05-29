# Day 12 — JavaScript & TypeScript Revision

**Topics:** Memoization · Cache strategies · LRU connection

---

## Table of Contents

1. [Memoization Fundamentals](#1-memoization-fundamentals)
2. [memoize Implementation](#2-memoize-implementation)
3. [Cache Key Strategies](#3-cache-key-strategies)
4. [Memoization vs Memo (React)](#4-memoization-vs-memo-react)
5. [LRU Preview (Machine Coding)](#5-lru-preview-machine-coding)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Memoization Fundamentals

### Definition

Cache **function results** by arguments to avoid recomputation.

```js
function slowSquare(n) {
  for (let i = 0; i < 1e6; i++) {}
  return n * n;
}
```

**Pure functions** only — same inputs → same output, no side effects.

### When to use

| Good | Bad |
|------|-----|
| Expensive pure fn (fib, dp) | Random / time-dependent |
| Repeated calls same args | Unbounded arg space (memory leak) |
| API response by id | Functions with side effects |

---

## 2. memoize Implementation

### Basic (single primitive arg)

```js
function memoize(fn) {
  const cache = new Map();
  return function (arg) {
    if (cache.has(arg)) return cache.get(arg);
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}

const fastFib = memoize(function fib(n) {
  if (n <= 1) return n;
  return fastFib(n - 1) + fastFib(n - 2);
});
```

### Generic — multiple args (JSON key)

```js
function memoize(fn, keyFn = (...args) => JSON.stringify(args)) {
  const cache = new Map();
  return function (...args) {
    const key = keyFn(...args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

### With TTL

```js
function memoizeTTL(fn, ttlMs) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    const hit = cache.get(key);
    if (hit && Date.now() - hit.t < ttlMs) return hit.v;
    const v = fn(...args);
    cache.set(key, { v, t: Date.now() });
    return v;
  };
}
```

### Bounded cache (LRU sketch)

```js
function memoizeLRU(fn, max = 100) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      const v = cache.get(key);
      cache.delete(key);
      cache.set(key, v); // refresh
      return v;
    }
    const result = fn(...args);
    cache.set(key, result);
    if (cache.size > max) {
      const oldest = cache.keys().next().value;
      cache.delete(oldest);
    }
    return result;
  };
}
```

---

## 3. Cache Key Strategies

| Strategy | When |
|----------|------|
| Primitive arg as key | Single number/string |
| `JSON.stringify(args)` | Simple serializable args |
| Custom `keyFn` | Objects — pick stable fields |
| `WeakMap` | Object identity cache (Day 13) |

```js
const memoById = memoize(fetchUser, (id) => String(id));
```

### Pitfall — object key by reference

```js
const cache = new Map();
cache.set({ a: 1 }, "x"); // new object each time — never hits
```

---

## 4. Memoization vs Memo (React)

| | JS memoization | `React.memo` |
|---|----------------|--------------|
| Caches | Function return values | Component render output |
| Scope | Any function | React components |
| Invalidation | Manual / LRU / TTL | Props shallow compare |

```tsx
const ExpensiveRow = React.memo(function Row({ item }: { item: Item }) {
  return <div>{item.name}</div>;
});
```

`useMemo` / `useCallback` — memoize **values/functions** inside component render cycle.

---

## 5. LRU Preview (Machine Coding)

**LRU** = evict **least recently used** when capacity exceeded — same Map order trick as memoizeLRU above.

Operations: `get`, `put` in O(1) with Map + doubly linked list (full impl in machine coding file).

---

## 6. Interview Quick Index

| Question | Section |
|----------|---------|
| What is memoization? | [§1](#1-memoization-fundamentals) |
| Implement memoize | [§2](#2-memoize-implementation) |
| JSON key pitfalls | [§3](#3-cache-key-strategies) |
| memoize vs React.memo | [§4](#4-memoization-vs-memo-react) |
| Bounded cache | [§2–§5](#5-lru-preview-machine-coding) |

---

## Day 12 Cheat Sheet

```
memoize  → Map cache; pure functions only
Keys     → JSON.stringify or custom keyFn
LRU      → Map insertion order; delete+set on access
React    → memo/useMemo/useCallback separate concerns
```

---

*End of Day 12 revision*
