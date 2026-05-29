# Day 6 — Saturday Revision (Week 1 Consolidation)

**Topics:** Closures Review · Promises Review · Generics Review · Implement `Promise.all`

---

## Table of Contents

1. [Closures — Quick Review](#1-closures--quick-review)
2. [Promises — Quick Review](#2-promises--quick-review)
3. [Generics — Quick Review](#3-generics--quick-review)
4. [Implement Promise.all from Scratch](#4-implement-promiseall-from-scratch)
5. [Implement Promise.allSettled & Promise.race](#5-implement-promiseallsettled--promiserace)
6. [Week 1 Concepts Integration](#6-week-1-concepts-integration)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Closures — Quick Review

### Definition (one line)

> Function + remembered outer variables (lexical environment).

### Must-know patterns

```js
// Counter
function createCounter(initial = 0) {
  let count = initial;
  return {
    increment: () => ++count,
    decrement: () => --count,
    get value() { return count; },
  };
}

// Memoization
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Module pattern
const calculator = (() => {
  let result = 0;
  return {
    add(x) { result += x; return this; },
    getResult() { return result; },
  };
})();
```

### Classic trap — loop + setTimeout

```js
// FIX with let (block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2
}

// FIX with IIFE
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i);
}
```

### Closure vs class private field

| Closure | `#private` field |
|---------|------------------|
| Works everywhere | ES2022+ |
| Per-instance via factory | Class syntax |
| Used in hooks pattern | OOP encapsulation |

---

## 2. Promises — Quick Review

### State machine

```
pending → fulfilled (resolve)
        → rejected  (reject)
```

### Chaining rules

1. `.then(fn)` returns new Promise
2. Return value → next `.then` receives it
3. Return Promise → waits for it
4. Throw → next `.catch` handles

### async/await equivalents

```js
// Promise chain
fetchUser(id)
  .then((user) => fetchOrders(user.id))
  .then(processOrders)
  .catch(handleError);

// async/await
async function load(id) {
  try {
    const user = await fetchUser(id);
    const orders = await fetchOrders(user.id);
    return processOrders(orders);
  } catch (err) {
    handleError(err);
  }
}
```

### Parallel vs sequential

```js
// Parallel — total time = max(individual)
const [a, b, c] = await Promise.all([fetchA(), fetchB(), fetchC()]);

// Sequential — total time = sum(individual)
const a = await fetchA();
const b = await fetchB(a);
const c = await fetchC(b);
```

### Microtask reminder

```js
Promise.resolve().then(() => console.log("micro"));
setTimeout(() => console.log("macro"), 0);
// micro → macro
```

---

## 3. Generics — Quick Review

### Core syntax

```ts
function identity<T>(x: T): T { return x; }
function first<T>(arr: T[]): T | undefined { return arr[0]; }
function merge<T, U>(a: T, b: U): T & U { return { ...a, ...b }; }
```

### Constraints

```ts
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}
```

### Generic interfaces

```ts
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}
```

### React generic component

```tsx
type SelectProps<T> = {
  options: T[];
  getLabel: (item: T) => string;
  onSelect: (item: T) => void;
};

function Select<T>({ options, getLabel, onSelect }: SelectProps<T>) {
  return (
    <select onChange={(e) => onSelect(options[Number(e.target.value)])}>
      {options.map((opt, i) => (
        <option key={i} value={i}>{getLabel(opt)}</option>
      ))}
    </select>
  );
}
```

### Utility types recap

| Type | Purpose |
|------|---------|
| `Partial<T>` | All optional |
| `Pick<T, K>` | Select keys |
| `Omit<T, K>` | Remove keys |
| `Record<K, V>` | Key-value map |
| `ReturnType<F>` | Function return type |
| `Parameters<F>` | Function param tuple |

---

## 4. Implement Promise.all from Scratch

### Requirements

- Takes iterable of Promises (or values)
- Returns single Promise
- Resolves to array of results **in order**
- Rejects immediately on first rejection
- Handles non-Promise values (wrap with `Promise.resolve`)

### Implementation

```js
function promiseAll(iterable) {
  return new Promise((resolve, reject) => {
    const items = Array.from(iterable);
    const results = new Array(items.length);
    let remaining = items.length;

    if (remaining === 0) {
      resolve(results);
      return;
    }

    items.forEach((item, index) => {
      Promise.resolve(item)
        .then((value) => {
          results[index] = value;
          remaining--;
          if (remaining === 0) resolve(results);
        })
        .catch(reject);
    });
  });
}
```

### TypeScript version

```ts
function promiseAll<T extends readonly unknown[] | []>(
  values: T
): Promise<{ [K in keyof T]: Awaited<T[K]> }> {
  return new Promise((resolve, reject) => {
    const items = Array.from(values) as unknown[];
    const results: unknown[] = new Array(items.length);
    let remaining = items.length;

    if (remaining === 0) {
      resolve([] as { [K in keyof T]: Awaited<T[K]> });
      return;
    }

    items.forEach((item, index) => {
      Promise.resolve(item)
        .then((value) => {
          results[index] = value;
          remaining--;
          if (remaining === 0) {
            resolve(results as { [K in keyof T]: Awaited<T[K]> });
          }
        })
        .catch(reject);
    });
  });
}
```

### Test cases

```js
// All resolve
promiseAll([1, Promise.resolve(2), Promise.resolve(3)])
  .then(console.log); // [1, 2, 3]

// First reject
promiseAll([Promise.resolve(1), Promise.reject("err"), Promise.resolve(3)])
  .catch(console.log); // "err"

// Empty
promiseAll([]).then(console.log); // []

// Order preserved despite timing
promiseAll([
  new Promise((r) => setTimeout(() => r("slow"), 200)),
  Promise.resolve("fast"),
]).then(console.log); // ["slow", "fast"]
```

### Interview talking points

| Point | Detail |
|-------|--------|
| Why `Promise.resolve(item)`? | Handles non-Promise values |
| Why `results[index]`? | Preserve order despite async timing |
| Empty array | Resolve immediately to `[]` |
| First reject | `.catch(reject)` on inner promise |
| vs Promise.allSettled | allSettled never rejects; waits for all |

---

## 5. Implement Promise.allSettled & Promise.race

### Promise.allSettled

```js
function promiseAllSettled(iterable) {
  return Promise.all(
    Array.from(iterable).map((item) =>
      Promise.resolve(item)
        .then((value) => ({ status: "fulfilled", value }))
        .catch((reason) => ({ status: "rejected", reason }))
    )
  );
}
```

### Promise.race

```js
function promiseRace(iterable) {
  return new Promise((resolve, reject) => {
    for (const item of iterable) {
      Promise.resolve(item).then(resolve, reject);
    }
  });
}
```

### Promise.any (bonus)

```js
function promiseAny(iterable) {
  return new Promise((resolve, reject) => {
    const errors = [];
    let remaining = 0;

    for (const item of iterable) {
      remaining++;
      Promise.resolve(item).then(resolve, (err) => {
        errors.push(err);
        remaining--;
        if (remaining === 0) {
          reject(new AggregateError(errors, "All promises rejected"));
        }
      });
    }

    if (remaining === 0) reject(new AggregateError([], "Empty iterable"));
  });
}
```

---

## 6. Week 1 Concepts Integration

### Day 1–5 map

| Day | JS/TS | Pattern |
|-----|-------|---------|
| 1 | Closures, event loop, `this`, TS basics | Stack/queue intro |
| 2 | Prototypes, union/intersection, type guards | Monotonic stack |
| 3 | Promises, async/await, generics | Queue patterns |
| 4 | Call stack, event loop deep, mapped types, keyof | Stack simulation |
| 5 | Debounce/throttle, conditional types, infer | Stack span, heap |

### Cross-topic interview question

**"Build a debounced search that cancels previous requests"**

Touches:
- Debounce (Day 5)
- Promises + AbortController (Day 3)
- Closures in hook (Day 1/6)
- Generics for typed results (Day 3/6)

```tsx
function useDebouncedSearch<T>(fetcher: (q: string) => Promise<T>, delay = 300) {
  const abortRef = useRef<AbortController>();
  // closure over abortRef + debounce timer
}
```

---

## 7. Interview Quick Index

| Question | Section |
|----------|---------|
| Closure definition + counter | [§1](#1-closures--quick-review) |
| setTimeout loop trap | [§1](#1-closures--quick-review) |
| Promise.all behavior | [§2](#2-promises--quick-review) |
| Microtask before macrotask | [§2](#2-promises--quick-review) |
| Generic constraint `extends` | [§3](#3-generics--quick-review) |
| Implement Promise.all | [§4](#4-implement-promiseall-from-scratch) |
| Promise.allSettled vs all | [§5](#5-implement-promiseallsettled--promiserace) |

---

## Day 6 Cheat Sheet (Saturday revision)

```
Closure      → fn + outer vars; module pattern, memoize
Promise.all  → all resolve or first reject; preserve order
Implement    → Promise.resolve each; results[i]; remaining counter
Generics     → <T>, extends constraint, generic interfaces
Microtasks   → before setTimeout; await continuation is microtask
Week 1       → stacks, queues, promises, debounce connect in machine coding
```

---

*End of Day 6 revision*
