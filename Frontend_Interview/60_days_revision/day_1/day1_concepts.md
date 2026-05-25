# Day 1 — JavaScript & TypeScript Revision

**Topics:** Closures · Currying · Execution Context & Event Loop · `this` · TypeScript basics

---

## Table of Contents

1. [Closures](#1-closures)
2. [Currying](#2-currying)
3. [Execution Context, Call Stack & Event Loop](#3-execution-context-call-stack--event-loop)
4. [`this` Keyword](#4-this-keyword)
5. [TypeScript Basics](#5-typescript-basics)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Closures

### Foundation — Lexical Scope

Inner functions access variables from **parent scope** (where code is **written**, not where it is called).

```js
function outer() {
  let a = 10;
  function inner() { console.log(a); }
  inner();
}
// 10
```

### Definition

> **Closure** = function + its **lexical environment** (remembered outer variables).

Happens when an inner function is **returned or passed** but still remembers outer variables after the outer function finishes.

```js
function outer() {
  let count = 0;
  return function inner() {
    count++;
    console.log(count);
  };
}
const fn = outer();
fn(); fn(); fn();  // 1, 2, 3
```

### How it works internally

When `outer()` ends, `inner` still references `x` → engine keeps `x` alive in **closure memory** (`[[Closure]]`).

### Real-world uses

| Use | Idea |
|-----|------|
| **Data hiding** | Private variables via returned object |
| **Counter / factory** | `createCounter()`, `multiply(x)` |
| **Memoization** | Cache inside closure |
| **Module pattern** | IIFE + returned API |

```js
function bankAccount(balance) {
  return {
    deposit(amt) { balance += amt; },
    withdraw(amt) { balance -= amt; },
  };
}
// balance is private — not accessible from outside
```

### setTimeout + `var` vs `let` (must know)

```js
// BUG — var: same i shared, loop ends at i=6
for (var i = 1; i <= 5; i++) {
  setTimeout(() => console.log(i), i * 1000);
}
// 6,6,6,6,6

// FIX 1 — let: new binding each iteration
for (let i = 1; i <= 5; i++) {
  setTimeout(() => console.log(i), i * 1000);
}
// 1,2,3,4,5

// FIX 2 — IIFE / wrapper (interview favorite)
for (var i = 1; i <= 5; i++) {
  (function (j) {
    setTimeout(() => console.log(j), j * 1000);
  })(i);
}
```

### Memoization (closure)

```js
function memoize(fn) {
  const cache = {};
  return function (n) {
    if (cache[n]) return cache[n];
    return (cache[n] = fn(n));
  };
}
```

### Pros & cons

| Pros | Cons |
|------|------|
| Privacy, factories, memoization | Extra memory if closures hold large data |
| Module pattern | Possible leaks if not cleaned up |

### Tricky outputs (revision)

| Code | Output | Why |
|------|--------|-----|
| `var z = x(); z();` where `x` sets `a=7`, returns `y`, then `a=100` | `100` | Closure holds **reference**, not copy |
| `for (var i=0; i<3; i++) setTimeout(...)` | `3,3,3` | Shared `var` |
| `for (let i=0; i<3; i++) setTimeout(...)` | `0,1,2` | Block scope per iteration |

**One-liner:** Function bundled with lexical environment; outer variables stay alive after outer returns.

---

## 2. Currying

### Definition

Transform `f(a, b, c)` into `f(a)(b)(c)` — one argument per nested function.

```js
const add = a => b => c => a + b + c;
add(1)(2)(3);  // 6
```

### Why use it

- **Reusability:** `const double = multiply(2); double(5) → 10`
- **Partial application** (some args fixed) vs currying (one at a time)

### Generic `curry()` (interview)

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...next) => curried(...args, ...next);
  };
}
// curriedSum(1)(2)(3) === curriedSum(1,2)(3) === 6
```

### Infinite currying (sketch)

```js
function add(a) {
  return function (b) {
    if (b !== undefined) return add(a + b);
    return a;
  };
}
add(1)(2)(3)(4)();  // 10
```

### Closure vs Currying

| Closure | Currying |
|---------|----------|
| Remembers outer scope | Splits multi-arg function |
| Data hiding | Reuses functions; uses closures internally |

**One-liner:** Currying = multi-arg function → chain of single-arg functions.

---

## 3. Execution Context, Call Stack & Event Loop

### Core facts

- JS is **single-threaded** — one call stack, one thing at a time.
- Async via **Web APIs** + **queues** + **event loop**.

### Execution Context (EC)

Environment where code runs: variables, functions, `this`, scope.

| Type | When |
|------|------|
| **GEC** | Program start (`window` / `global`) |
| **FEC** | Each function call |

**Two phases:** (1) Memory creation (hoisting) → (2) Code execution.

| Hoisting | Memory phase result |
|----------|---------------------|
| `var` | `undefined` |
| `function` declaration | Full function |
| `let` / `const` | TDZ until declaration line |

### Call stack

LIFO — tracks current function. Overflow = too much recursion.

```
one() → two() → three() → pop back
```

### Web APIs (browser)

`setTimeout`, `fetch`, DOM events — **not** in JS engine; browser handles, then queues callback.

### Queues & priority

```
1. Call stack (sync first)
2. Microtask queue (ALL drained)
3. Macrotask queue (ONE per turn)
```

| Microtask | Macrotask |
|-----------|-----------|
| `Promise.then`, `queueMicrotask`, `async/await` resume | `setTimeout`, `setInterval`, DOM events |

### Classic output

```js
console.log("Start");
setTimeout(() => console.log("Timeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
console.log("End");
// Start → End → Promise → Timeout
```

### `async/await`

`await` pauses function; continuation goes to **microtask queue**.

```js
async function test() {
  console.log(1);
  await Promise.resolve();
  console.log(2);
}
console.log(3);
test();
console.log(4);
// 3, 1, 4, 2
```

### Advanced trick

```js
setTimeout(() => console.log(1), 0);
Promise.resolve().then(() => console.log(2));
Promise.resolve().then(() => {
  setTimeout(() => console.log(3), 0);
});
Promise.resolve().then(() => console.log(4));
console.log(5);
// 5, 2, 4, 1, 3
```

### Golden rules

1. Sync runs first until stack empty.
2. **All** microtasks before next macrotask.
3. `setTimeout(0)` ≠ immediate — after stack + microtasks.
4. Infinite microtasks can **starve** macrotasks.

### Node.js (extra)

- `process.nextTick` — before Promise microtasks.
- `setImmediate` — check phase (after I/O).

**One-liner:** Event loop pushes queued callbacks when stack is empty; promises beat timers.

---

## 4. `this` Keyword

### Rule

> `this` is set at **runtime** by **how** the function is called — not where it is defined.

### Binding types (priority high → low)

1. **new** → new object  
2. **Explicit** — `call` / `apply` / `bind`  
3. **Implicit** — `obj.method()` → `this = obj`  
4. **Default** — standalone call → `window` (non-strict) or `undefined` (strict)

### Quick reference

| Context | Non-strict (browser) | Strict |
|---------|----------------------|--------|
| Global | `window` | `undefined` (modules too) |
| Normal function `fn()` | `window` | `undefined` |
| `obj.method()` | `obj` | `obj` |
| Arrow function | Lexical `this` from parent | Same |
| `new Fn()` | new instance | new instance |
| Event listener (regular fn) | DOM element | DOM element |

### Arrow vs normal

```js
const obj = {
  name: "JS",
  normal() { console.log(this.name); },      // "JS"
  arrow: () => console.log(this.name),       // undefined (outer = window)
};
```

Arrows: **no** own `this`, `arguments`, `new` — cannot use `call`/`apply`/`bind` to change `this`.

### `call` / `apply` / `bind`

```js
greet.call(user, "Kochi");           // run now, comma args
greet.apply(user, ["Kochi"]);        // run now, array args
const fn = greet.bind(user, "Kochi"); // new fn, run later
```

### Common traps

```js
const fn = obj.show; fn();           // lost `this` → window/undefined
const fn = obj.show.bind(obj); fn(); // fixed

obj.show()();                        // returned normal fn → loses `this`
obj.show()();                        // returned arrow → keeps `this`
```

### `new` keyword steps

1. Create empty object  
2. Link prototype  
3. `this` = that object  
4. Run constructor  
5. Return object (unless constructor returns other object)

### Strict mode

```js
"use strict";
function User(name) { this.name = name; }
User("JS");  // TypeError — this is undefined, not window
```

**One-liner:** Normal fn → caller decides `this`; arrow → parent scope; `bind` fixes lost context.

---

## 5. TypeScript Basics

### What it is

**Superset of JS** with static types → compiles to JS (`tsc`). Catches errors at **compile time**.

### Basic types

| Type | Example |
|------|---------|
| `number`, `string`, `boolean` | Primitives |
| `any` | No check (avoid) |
| `unknown` | Safe `any` — must narrow before use |
| `void` | No return |
| `never` | Never returns (throw / infinite loop) |
| `number[]` / `Array<number>` | Arrays |
| `[string, number]` | Tuple — fixed length/order |

### `any` vs `unknown`

```ts
let u: unknown = "hi";
// u.toUpperCase();  // ERROR
if (typeof u === "string") u.toUpperCase();  // OK
```

### Union & literal

```ts
let id: string | number;
let dir: "left" | "right";
```

### Interface vs type alias

| Interface | Type alias |
|-----------|------------|
| Objects, extend (`extends`), declaration merging | Unions, primitives, flexible |
| Preferred for object shapes | `type ID = string \| number` |

```ts
interface User {
  name: string;
  age?: number;           // optional
  readonly id: number;   // immutable
}
```

### Functions

```ts
function add(a: number, b: number): number { return a + b; }
function greet(name: string = "Guest"): void { }
function sum(...nums: number[]): number { }
```

### Generics

```ts
function identity<T>(arg: T): T { return arg; }
function getFirst<T>(arr: T[]): T { return arr[0]; }
```

### Classes

| Modifier | Access |
|----------|--------|
| `public` | Everywhere (default) |
| `private` | Class only |
| `protected` | Class + subclasses |

### Strict mode (`tsconfig`)

```json
{ "compilerOptions": { "strict": true } }
```

Enables: `strictNullChecks`, `noImplicitAny`, safer functions, etc.

- Without strict: `let s: string = null` may be allowed.  
- With strict: must handle `null` / `undefined`.

### Utility types (know names)

| Utility | Effect |
|---------|--------|
| `Partial<T>` | All props optional |
| `Readonly<T>` | Immutable |
| `Pick<T, K>` | Subset of keys |
| `Omit<T, K>` | Remove keys |

### Type assertion

```ts
(value as string).toUpperCase();
```

### Enum

```ts
enum Role { ADMIN, USER }  // 0, 1 by default
enum Status { OK = 200, NOT_FOUND = 404 }
```

### Interview snippets

| Question | Answer |
|----------|--------|
| TS vs JS | Static vs dynamic; compile-time vs runtime errors |
| `interface` vs `type` | Interface for objects/extend; type for unions |
| `never` vs `void` | Never returns vs returns nothing |
| Prefer `unknown` over `any` | Forces type narrowing |

**One-liner:** TS = JS + static types; enable `strict`; use `interface` for objects, `generics` for reuse.

---

## 6. Interview Quick Index

Questions map to sections above.

### Closures & Currying

| Question | Section |
|----------|---------|
| What is a closure? | [§1](#1-closures) |
| Lexical scope vs closure | [§1](#1-closures) |
| setTimeout prints 6 six times | [§1 — var/let](#1-closures) |
| Data hiding / module pattern | [§1](#1-closures) |
| Memoization with closure | [§1](#1-closures) |
| What is currying? `curry()` impl | [§2](#2-currying) |
| Currying vs partial application | [§2](#2-currying) |
| Closure vs currying | [§2](#2-currying) |

### Event loop

| Question | Section |
|----------|---------|
| Single-threaded? How async works? | [§3](#3-execution-context-call-stack--event-loop) |
| Execution context phases | [§3](#3-execution-context-call-stack--event-loop) |
| Microtask vs macrotask | [§3](#3-execution-context-call-stack--event-loop) |
| Start, End, Promise, Timeout order | [§3 — classic](#3-execution-context-call-stack--event-loop) |
| `async/await` output order | [§3](#3-execution-context-call-stack--event-loop) |
| Why `setTimeout(0)` is not instant | [§3](#3-execution-context-call-stack--event-loop) |

### `this`

| Question | Section |
|----------|---------|
| How is `this` determined? | [§4](#4-this-keyword) |
| Arrow vs normal `this` | [§4](#4-this-keyword) |
| `call` / `apply` / `bind` | [§4](#4-this-keyword) |
| `const fn = obj.method; fn()` | [§4 — traps](#4-this-keyword) |
| Strict mode + constructor | [§4](#4-this-keyword) |
| Binding priority | [§4](#4-this-keyword) |

### TypeScript

| Question | Section |
|----------|---------|
| What is TypeScript? Why use it? | [§5](#5-typescript-basics) |
| `any` vs `unknown` | [§5](#5-typescript-basics) |
| `interface` vs `type` | [§5](#5-typescript-basics) |
| Strict mode | [§5](#5-typescript-basics) |
| Generics | [§5](#5-typescript-basics) |
| Utility types | [§5](#5-typescript-basics) |

---

## Day 1 Cheat Sheet (30-second recall)

```
Closure     → fn + remembered outer vars
Currying    → f(a,b,c) → f(a)(b)(c)
Event loop  → stack → all microtasks → one macrotask
this        → who called the function (except arrow = lexical)
TS          → types at compile time; strict + interface + generics
```

---

*End of Day 1 revision*
