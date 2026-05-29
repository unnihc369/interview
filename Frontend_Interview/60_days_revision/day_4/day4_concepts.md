# Day 4 — JavaScript & TypeScript Revision

**Topics:** Event Loop Deep Dive · Call Stack · Callback Queue · Mapped Types · `keyof`

---

## Table of Contents

1. [Call Stack — How JS Executes](#1-call-stack--how-js-executes)
2. [Event Loop Architecture](#2-event-loop-architecture)
3. [Callback Queue (Macrotask Queue)](#3-callback-queue-macrotask-queue)
4. [Microtasks vs Macrotasks — Deep Dive](#4-microtasks-vs-macrotasks--deep-dive)
5. [TypeScript Mapped Types](#5-typescript-mapped-types)
6. [The `keyof` Operator](#6-the-keyof-operator)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Call Stack — How JS Executes

### LIFO structure

The **call stack** tracks function execution. Last function **pushed** is first **popped**.

```js
function third() { console.log("third"); }
function second() { third(); }
function first() { second(); }

first();
// Stack: first → second → third → (pop all)
```

### Stack overflow

```js
function recurse() { recurse(); }
recurse(); // RangeError: Maximum call stack size exceeded
```

### Execution context

Each stack frame has:
- Variable environment (local vars, params)
- `this` binding
- Reference to outer environment (closure)

```js
function outer(x) {
  const y = 10;
  function inner(z) {
    return x + y + z; // closure over x, y
  }
  return inner;
}
```

### Sync code runs to completion

JavaScript is **single-threaded**. One call stack. Long sync loop blocks UI and timers.

```js
console.log("start");
for (let i = 0; i < 1e9; i++) { /* heavy */ }
console.log("end");
// Timers won't fire until loop finishes
```

---

## 2. Event Loop Architecture

### Components (browser / Node.js similar concept)

```
┌───────────────────────────┐
│        Call Stack         │  ← sync code runs here
└───────────────────────────┘
            ↓
┌───────────────────────────┐
│     Microtask Queue       │  ← Promises, queueMicrotask
└───────────────────────────┘
            ↓
┌───────────────────────────┐
│   Macrotask Queue         │  ← setTimeout, I/O, DOM events
└───────────────────────────┘
            ↓
         (repeat)
```

### Loop algorithm (simplified)

1. Execute sync code until call stack empty
2. Run **all** microtasks until microtask queue empty
3. Render (browser) if needed
4. Dequeue **one** macrotask → push to call stack
5. Go to step 1

### Node.js differences (bonus)

| Browser | Node.js |
|---------|---------|
| Microtasks = Promise callbacks | `process.nextTick` runs before Promise microtasks |
| Macrotasks = setTimeout | `setImmediate`, I/O phases in libuv |

---

## 3. Callback Queue (Macrotask Queue)

### setTimeout / setInterval

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
console.log("C");
// A, C, B
```

Even `0ms` delay → callback goes to macrotask queue, runs **after** current sync + microtasks.

### DOM event handlers

Click handlers are macrotasks (task queue in HTML spec).

```js
button.addEventListener("click", () => console.log("clicked"));
// Runs after current stack clears
```

### MessageChannel / postMessage

Also macrotask — used by libraries to defer work.

### requestAnimationFrame

**Not** a macrotask in the same sense — runs before paint, after microtasks in browser.

```js
requestAnimationFrame(() => console.log("rAF"));
Promise.resolve().then(() => console.log("micro"));
// micro → rAF (before paint) → ...
```

---

## 4. Microtasks vs Macrotasks — Deep Dive

### Hard interview question

```js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve()
  .then(() => {
    console.log("3");
    Promise.resolve().then(() => console.log("4"));
  })
  .then(() => console.log("5"));

console.log("6");

// Output: 1, 6, 3, 4, 5, 2
```

### Step-by-step

1. Sync: `1`, `6`
2. Microtasks: `3` (first `.then`)
3. Microtasks: `4` (nested Promise inside `3`)
4. Microtasks: `5` (chained `.then` after `3`'s handler returns)
5. Macrotask: `2` (setTimeout)

### async/await is microtask-based

```js
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
console.log("script start");
async1();
console.log("script end");

// script start → async1 start → async2 → script end → async1 end
```

### queueMicrotask

```js
queueMicrotask(() => console.log("micro"));
Promise.resolve().then(() => console.log("promise"));
// Order: micro, promise (FIFO within microtask queue)
```

### Blocking the event loop

```js
// BAD — starves macrotasks
function block() {
  const start = Date.now();
  while (Date.now() - start < 5000) {}
  Promise.resolve().then(block); // microtask loop
}
```

---

## 5. TypeScript Mapped Types

### Definition

Transform properties of an existing type by **mapping over keys**.

```ts
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

type Partial<T> = {
  [K in keyof T]?: T[K];
};
```

### Custom mapped type

```ts
type User = { id: string; name: string; email: string };

type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type NullableUser = Nullable<User>;
// { id: string | null; name: string | null; email: string | null }
```

### Key remapping (TS 4.1+)

```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getId: () => string; getName: () => string; getEmail: () => string }
```

### Filter keys with `as` + conditional

```ts
type PickByType<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};

type UserStrings = PickByType<User, string>;
// { id: string; name: string; email: string }
```

### Built-in mapped utility types

| Utility | Effect |
|---------|--------|
| `Partial<T>` | All optional |
| `Required<T>` | All required |
| `Readonly<T>` | All readonly |
| `Pick<T, K>` | Subset of keys |
| `Omit<T, K>` | Exclude keys |
| `Record<K, V>` | Object with keys K and values V |

```ts
type Roles = "admin" | "user" | "guest";
type RolePermissions = Record<Roles, string[]>;
// { admin: string[]; user: string[]; guest: string[] }
```

---

## 6. The `keyof` Operator

### Extract union of keys

```ts
type User = { id: string; name: string; age: number };
type UserKeys = keyof User; // "id" | "name" | "age"
```

### Safe property access

```ts
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: "1", name: "Alice", age: 30 };
getProp(user, "name");  // string
// getProp(user, "foo"); // Error
```

### keyof with indexed access

```ts
type User = { id: string; profile: { bio: string } };
type ProfileKeys = keyof User["profile"]; // "bio"
```

### keyof with unions

```ts
type A = { a: string };
type B = { b: number };
type Keys = keyof (A | B); // "a" | "b" (intersection of keys in union)
```

### Template literal keys

```ts
type EventNames = "click" | "focus";
type Handlers = {
  [K in EventNames as `on${Capitalize<K>}`]: () => void;
};
// { onClick: () => void; onFocus: () => void }
```

### Practical React example

```ts
type FormState = { email: string; password: string };

function handleChange<K extends keyof FormState>(
  key: K,
  value: FormState[K]
) {
  setForm((prev) => ({ ...prev, [key]: value }));
}

handleChange("email", "a@b.com");     // OK
// handleChange("email", 123);        // Error
```

---

## 6b. HTTP & Browser Networking (Overview)

Pairs with event loop — fetch responses arrive as **macrotasks/microtasks**.

| Topic | Detail |
|-------|--------|
| **CORS** | Browser blocks cross-origin reads unless server allows |
| **Status codes** | 401 auth, 403 forbidden, 404 missing, 429 rate limit |
| **AbortController** | Cancel fetch on unmount / new query (Day 3, 5) |
| **Credentials** | `credentials: "include"` sends cookies — needs CORS |

Full guide: [supplements/sde1_frontend_networking.md](../supplements/sde1_frontend_networking.md)  
Security (XSS, CSRF, tokens): [supplements/sde1_frontend_security.md](../supplements/sde1_frontend_security.md)

---

### Interview snippets

| Question | Answer |
|----------|--------|
| Call stack overflow cause | Infinite recursion without base case |
| setTimeout(0) delay? | Waits for sync + all microtasks |
| Mapped type syntax | `[K in keyof T]: ...` |
| keyof purpose | Union of object keys for type-safe access |
| Pick vs Omit | Pick selects keys; Omit excludes keys |

---

## 7. Interview Quick Index

### Event Loop

| Question | Section |
|----------|---------|
| Call stack LIFO | [§1](#1-call-stack--how-js-executes) |
| Event loop steps | [§2](#2-event-loop-architecture) |
| Callback/macrotask queue | [§3](#3-callback-queue-macrotask-queue) |
| 1,6,3,4,5,2 output | [§4](#4-microtasks-vs-macrotasks--deep-dive) |
| async/await order | [§4](#4-microtasks-vs-macrotasks--deep-dive) |

### TypeScript

| Question | Section |
|----------|---------|
| Mapped types | [§5](#5-typescript-mapped-types) |
| Key remapping `as` | [§5](#5-typescript-mapped-types) |
| keyof operator | [§6](#6-the-keyof-operator) |
| getProp generic pattern | [§6](#6-the-keyof-operator) |

---

## Day 4 Cheat Sheet (30-second recall)

```
Call stack   → LIFO; sync code; overflow = infinite recursion
Event loop   → stack empty → all microtasks → one macrotask → repeat
Macrotask    → setTimeout, I/O, DOM events
Microtask    → Promise.then, queueMicrotask, await continuation
Mapped type  → [K in keyof T]: transform T[K]
keyof T      → union of property names; use with extends for safe access
```

---

*End of Day 4 revision*
