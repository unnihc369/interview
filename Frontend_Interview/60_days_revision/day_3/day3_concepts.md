# Day 3 — JavaScript & TypeScript Revision

**Topics:** Promises · async/await · Microtask Queue · TypeScript Generics (Basics)

---

## Table of Contents

1. [Promises — Foundation](#1-promises--foundation)
2. [Promise Chaining & Error Handling](#2-promise-chaining--error-handling)
3. [async/await](#3-asyncawait)
4. [Microtask Queue vs Macrotask](#4-microtask-queue-vs-macrotask)
5. [TypeScript Generics — Basics](#5-typescript-generics--basics)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Promises — Foundation

### Three states

| State | Meaning |
|-------|---------|
| **pending** | Initial; neither fulfilled nor rejected |
| **fulfilled** | Operation completed successfully |
| **rejected** | Operation failed |

Once settled (fulfilled/rejected), a Promise **cannot change state**.

### Creating a Promise

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("done"), 1000);
});

promise.then((value) => console.log(value)); // "done" after 1s
```

### Static helpers (must know)

```js
Promise.resolve(42);           // fulfilled with 42
Promise.reject(new Error("x")); // rejected

Promise.all([p1, p2, p3]);     // all succeed or first reject
Promise.allSettled([p1, p2]);  // wait for all; never rejects
Promise.race([p1, p2]);        // first settle wins
Promise.any([p1, p2]);         // first fulfill wins; reject if all fail
```

### Promise.all behavior

```js
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
Promise.all([p1, p2]).then(([a, b]) => console.log(a, b)); // 1 2

// One rejection → entire Promise.all rejects
Promise.all([
  Promise.resolve(1),
  Promise.reject("fail"),
]).catch((err) => console.log(err)); // "fail"
```

### Promise.allSettled (ES2020)

```js
Promise.allSettled([
  Promise.resolve("ok"),
  Promise.reject("err"),
]).then(console.log);
// [
//   { status: 'fulfilled', value: 'ok' },
//   { status: 'rejected', reason: 'err' }
// ]
```

---

## 2. Promise Chaining & Error Handling

### `.then` returns a new Promise

```js
Promise.resolve(1)
  .then((x) => x + 1)       // 2
  .then((x) => x * 3)       // 6
  .then(console.log);       // 6
```

### Returning Promise from `.then` flattens

```js
Promise.resolve(1)
  .then((x) => fetch(`/api/${x}`))  // returns Promise<Response>
  .then((res) => res.json())
  .then(console.log);
```

### Error propagation

```js
Promise.reject("oops")
  .then(() => console.log("skipped"))
  .catch((err) => console.log(err))  // "oops"
  .then(() => console.log("recovered")); // runs after catch
```

### `.finally` — cleanup regardless of outcome

```js
fetch("/api/data")
  .then((r) => r.json())
  .catch(console.error)
  .finally(() => hideSpinner());
```

### Anti-pattern: nested `.then` (callback hell in Promise form)

```js
// BAD
getUser(id).then((user) => {
  getOrders(user.id).then((orders) => {
    getDetails(orders[0].id).then(console.log);
  });
});

// GOOD — chain or async/await
getUser(id)
  .then((user) => getOrders(user.id))
  .then((orders) => getDetails(orders[0].id))
  .then(console.log);
```

---

## 3. async/await

### Syntax sugar over Promises

```js
async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error("HTTP " + res.status);
  return res.json();
}
```

### async function always returns a Promise

```js
async function getValue() {
  return 42;
}
getValue().then(console.log); // 42
```

### Error handling with try/catch

```js
async function loadData() {
  try {
    const user = await fetchUser(1);
    const orders = await fetchOrders(user.id);
    return orders;
  } catch (err) {
    console.error("Failed:", err);
    return [];
  }
}
```

### Sequential vs parallel

```js
// Sequential — slower (B waits for A)
const a = await fetchA();
const b = await fetchB(a.id);

// Parallel — faster
const [users, products] = await Promise.all([
  fetchUsers(),
  fetchProducts(),
]);
```

### await in loops

```js
// Sequential processing
for (const id of ids) {
  const result = await fetchItem(id);
  results.push(result);
}

// Parallel (careful with rate limits)
const results = await Promise.all(ids.map((id) => fetchItem(id)));
```

### Top-level await (ES modules)

```js
// module.mjs
const config = await fetch("/config.json").then((r) => r.json());
export default config;
```

---

## 4. Microtask Queue vs Macrotask

### Event loop order (critical for interviews)

1. Run **sync code** (call stack)
2. Drain **all microtasks** (Promise callbacks, queueMicrotask)
3. Run **one macrotask** (setTimeout, setInterval, I/O)
4. Repeat

### Classic output question

```js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

console.log("4");

// Output: 1, 4, 3, 2
```

**Why?** Sync first (1, 4). Microtasks before next macrotask (3). Then setTimeout (2).

### Microtask sources

| Source | Example |
|--------|---------|
| Promise `.then/.catch/.finally` | `Promise.resolve().then(fn)` |
| `queueMicrotask` | `queueMicrotask(() => {})` |
| `MutationObserver` | DOM mutation callbacks |
| `await` | Continuation after await is microtask |

### Macrotask sources

| Source | Example |
|--------|---------|
| `setTimeout` / `setInterval` | `setTimeout(fn, 0)` |
| I/O callbacks | `fs.readFile` |
| UI rendering | Browser paint cycles |

### Nested microtasks

```js
Promise.resolve()
  .then(() => {
    console.log("A");
    Promise.resolve().then(() => console.log("B"));
  })
  .then(() => console.log("C"));

// A, B, C — inner microtask runs before next .then in same chain turn
```

### async/await ordering

```js
async function foo() {
  console.log("start");
  await Promise.resolve();
  console.log("end");
}
foo();
console.log("after");

// start → after → end
// await splits function: sync part runs, then microtask for rest
```

### Starvation risk

Microtasks run until queue empty **before** any macrotask. Infinite microtask loop blocks timers.

```js
function loop() {
  Promise.resolve().then(loop);
}
loop();
setTimeout(() => console.log("never runs first"), 0);
```

---

## 5. TypeScript Generics — Basics

### Why generics?

Reuse logic across types while keeping **type safety**.

```ts
// Without generics — loses type info
function identityAny(arg: any): any { return arg; }

// With generics — preserves type
function identity<T>(arg: T): T { return arg; }

const num = identity(42);      // number
const str = identity("hello"); // string
```

### Generic functions

```ts
function getFirst<T>(arr: T[]): T | undefined {
  return arr[0];
}

getFirst([1, 2, 3]);     // number | undefined
getFirst(["a", "b"]);    // string | undefined
```

### Multiple type parameters

```ts
function pair<A, B>(a: A, b: B): [A, B] {
  return [a, b];
}
const p = pair("name", 42); // [string, number]
```

### Generic constraints

```ts
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(arg: T): number {
  return arg.length;
}

logLength("hello");  // OK
logLength([1, 2, 3]); // OK
// logLength(42);     // Error — number has no length
```

### Generic interfaces

```ts
interface ApiResponse<T> {
  data: T;
  status: number;
  error?: string;
}

type UserResponse = ApiResponse<User[]>;
type ProductResponse = ApiResponse<Product>;
```

### Generic classes

```ts
class Stack<T> {
  private items: T[] = [];
  push(item: T) { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}

const numStack = new Stack<number>();
numStack.push(1);
```

### Default type parameters

```ts
interface Paginated<T = unknown> {
  items: T[];
  page: number;
  total: number;
}

const page: Paginated = { items: [], page: 1, total: 0 };
const userPage: Paginated<User> = { items: [user], page: 1, total: 1 };
```

### keyof with generics (preview for Day 4)

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };
getProperty(user, "name"); // string
// getProperty(user, "email"); // Error
```

### React generic component pattern

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
};

function List<T>({ items, renderItem }: ListProps<T>) {
  return <ul>{items.map((item) => <li key={String(item)}>{renderItem(item)}</li>)}</ul>;
}
```

### Interview snippets

| Question | Answer |
|----------|--------|
| Promise vs async/await | Same underlying model; await is readable sync-style |
| Microtask vs macrotask | Promises before setTimeout |
| When Promise.all fails? | First rejection; others may still complete |
| What is `<T>`? | Type parameter — placeholder filled at call site |
| `extends` in generics | Constraint — T must satisfy shape |
| Generic vs `any` | Generic preserves type; any opts out |

---

## 5b. AbortController & Fetch Cancellation

When building `useQueue` or search — cancel stale requests.

```ts
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/data`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setData)
    .catch((err) => {
      if (err.name !== "AbortError") setError(err);
    });

  return () => controller.abort();
}, [deps]);
```

Full networking guide: [supplements/sde1_frontend_networking.md](../supplements/sde1_frontend_networking.md)

---

## 6. Interview Quick Index

### Promises & async

| Question | Section |
|----------|---------|
| Promise states | [§1](#1-promises--foundation) |
| Promise.all vs allSettled | [§1](#1-promises--foundation) |
| Error handling in chains | [§2](#2-promise-chaining--error-handling) |
| async/await error handling | [§3](#3-asyncawait) |
| Sequential vs parallel await | [§3](#3-asyncawait) |

### Event loop

| Question | Section |
|----------|---------|
| 1, 4, 3, 2 output order | [§4](#4-microtask-queue-vs-macrotask) |
| What runs before setTimeout(0)? | [§4](#4-microtask-queue-vs-macrotask) |
| await execution split | [§4](#4-microtask-queue-vs-macrotask) |

### Generics

| Question | Section |
|----------|---------|
| Why use generics? | [§5](#5-typescript-generics--basics) |
| Generic constraints | [§5](#5-typescript-generics--basics) |
| Generic interfaces/classes | [§5](#5-typescript-generics--basics) |

---

## Day 3 Cheat Sheet (30-second recall)

```
Promise       → pending | fulfilled | rejected
async/await   → syntax sugar; try/catch for errors
Microtasks    → Promise callbacks, queueMicrotask — run before macrotasks
Macrotasks    → setTimeout, I/O — one per loop turn
Generics <T>  → reusable typed code; extends for constraints
Parallel      → Promise.all([...]); Sequential → await in loop
```

---

*End of Day 3 revision*
