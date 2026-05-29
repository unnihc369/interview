# Day 17 — Symbol & Iterators

**Week 3 — Two Pointers** · **Topics:** `Symbol` · Well-known symbols · Iterators · Iterables · `for...of`

---

## Table of Contents

1. [Symbol Basics](#1-symbol-basics)
2. [Well-Known Symbols](#2-well-known-symbols)
3. [Iterator Protocol](#3-iterator-protocol)
4. [Iterable Protocol & for...of](#4-iterable-protocol--forof)
5. [Custom Iterators in UI](#5-custom-iterators-in-ui)
6. [Generators Preview (Day 18)](#6-generators-preview-day-18)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Symbol Basics

### What is a Symbol?

A **Symbol** is a primitive type (`Symbol()`) that produces a **guaranteed unique** value — ideal for object keys that won't collide.

```js
const id1 = Symbol("id");
const id2 = Symbol("id");
id1 === id2; // false — description is NOT part of identity
```

### Why not string keys?

```js
const PRIVATE = "private";
const obj = {
  [PRIVATE]: 42,
  private: "collision risk with future props",
};
```

Symbols are **not** enumerable in `for...in`, `Object.keys`, or spread — good for "hidden" metadata.

### Symbol.for / Symbol.keyFor (global registry)

```js
const a = Symbol.for("app.id");
const b = Symbol.for("app.id");
a === b; // true — same global symbol

Symbol.keyFor(a); // "app.id"
```

Use `Symbol.for` when plugins/modules need a **shared** key across bundles.

### Object property keys

```js
const KEY = Symbol("meta");
const user = { name: "Ada", [KEY]: { version: 1 } };

Object.getOwnPropertySymbols(user); // [Symbol(meta)]
Object.keys(user); // ["name"]
```

---

## 2. Well-Known Symbols

Built-in hooks on `Symbol`:

| Symbol | Purpose |
|--------|---------|
| `Symbol.iterator` | Default iterator (`for...of`) |
| `Symbol.asyncIterator` | Async iteration |
| `Symbol.toStringTag` | `[object Foo]` branding |
| `Symbol.hasInstance` | Custom `instanceof` |
| `Symbol.toPrimitive` | Coercion in `+`, `String()` |

### Symbol.toStringTag

```js
class Queue {
  get [Symbol.toStringTag]() {
    return "Queue";
  }
}
Object.prototype.toString.call(new Queue()); // "[object Queue]"
```

### Symbol.toPrimitive

```js
const money = {
  amount: 100,
  currency: "INR",
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.amount;
    return `${this.amount} ${this.currency}`;
  },
};
+money;        // 100
String(money); // "100 INR"
```

---

## 3. Iterator Protocol

An **iterator** is an object with a `next()` method returning `{ value, done }`.

```js
function createRangeIterator(start, end) {
  let current = start;
  return {
    next() {
      if (current <= end) {
        return { value: current++, done: false };
      }
      return { value: undefined, done: true };
    },
  };
}

const it = createRangeIterator(1, 3);
it.next(); // { value: 1, done: false }
it.next(); // { value: 2, done: false }
it.next(); // { value: 3, done: false }
it.next(); // { value: undefined, done: true }
```

### Manual consumption

```js
let result = it.next();
while (!result.done) {
  console.log(result.value);
  result = it.next();
}
```

### Built-in iterators

| Type | Iterator yields |
|------|-----------------|
| Array | indices 0..n-1 values |
| Map | `[key, value]` pairs |
| Set | values |
| String | Unicode code units (surrogate pairs caveat) |

---

## 4. Iterable Protocol & for...of

An object is **iterable** if it has `[Symbol.iterator]()` returning an iterator.

```js
const range = {
  from: 1,
  to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const end = this.to;
    return {
      next: () =>
        current <= end
          ? { value: current++, done: false }
          : { value: undefined, done: true },
    };
  },
};

for (const n of range) console.log(n); // 1 2 3 4 5
[...range]; // [1,2,3,4,5]
```

### Spread & destructuring require iterable

```js
const [a, b, ...rest] = range;
```

### Map / Set iteration order

- **Map:** insertion order of keys
- **Set:** insertion order of values

```js
const m = new Map([["b", 2], ["a", 1]]);
for (const [k] of m) console.log(k); // b, a
```

---

## 5. Custom Iterators in UI

### Paginated list iterator (infinite scroll mental model)

```js
function createPageIterator(fetchPage) {
  let page = 0;
  let buffer = [];
  let index = 0;

  return {
    async next() {
      if (index >= buffer.length) {
        buffer = await fetchPage(page++);
        index = 0;
        if (!buffer.length) return { value: undefined, done: true };
      }
      return { value: buffer[index++], done: false };
    },
    [Symbol.asyncIterator]() {
      return this;
    },
  };
}
```

### Triplet-sum game iterator (Day 17 MC tie-in)

Yield all unique triplets from sorted array for a game board:

```js
function* tripletIterator(nums) {
  const sorted = [...nums].sort((a, b) => a - b);
  const n = sorted.length;
  for (let i = 0; i < n - 2; i++) {
    if (i > 0 && sorted[i] === sorted[i - 1]) continue;
    let l = i + 1, r = n - 1;
    while (l < r) {
      const sum = sorted[i] + sorted[l] + sorted[r];
      if (sum === 0) {
        yield [sorted[i], sorted[l], sorted[r]];
        while (l < r && sorted[l] === sorted[l + 1]) l++;
        while (l < r && sorted[r] === sorted[r - 1]) r--;
        l++; r--;
      } else if (sum < 0) l++;
      else r--;
    }
  }
}
```

---

## 6. Generators Preview (Day 18)

Generators (`function*`) are syntactic sugar over iterators — return `{ value, done }` via `yield`.

```js
function* ids() {
  let n = 0;
  while (true) yield Symbol(`id-${++n}`);
}
```

Symbols + iterators = unique React keys from generator (anti-pattern for production — use stable IDs).

---

## 7. Interview Quick Index

| Question | Section |
|----------|---------|
| Symbol vs string key | [§1](#1-symbol-basics) — uniqueness, non-enumerable |
| `Symbol.for` vs `Symbol()` | [§1](#1-symbol-basics) |
| Iterator vs iterable | [§3](#3-iterator-protocol), [§4](#4-iterable-protocol--forof) |
| What makes array iterable? | `[Symbol.iterator]` on Array.prototype |
| Spread on Map | [§4](#4-iterable-protocol--forof) — yields entries |

---

## Day 17 Cheat Sheet

```
Symbol()       → unique key; not in Object.keys / for...in
Symbol.for(k)  → global shared symbol
Iterator       → { next() → { value, done } }
Iterable       → has [Symbol.iterator]()
for...of       → calls iterator until done
Map/Set        → insertion order when iterating
```

---

*End of Day 17 concepts*
