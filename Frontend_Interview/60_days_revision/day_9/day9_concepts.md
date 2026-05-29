# Day 9 — JavaScript & TypeScript Revision

**Topics:** Currying · Partial application · Difference & use cases

---

## Table of Contents

1. [Currying](#1-currying)
2. [Partial Application](#2-partial-application)
3. [Currying vs Partial Application](#3-currying-vs-partial-application)
4. [Advanced Patterns](#4-advanced-patterns)
5. [TypeScript](#5-typescript)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Currying

### Definition

Transform `f(a, b, c)` into `f(a)(b)(c)` — **one argument per function call**.

```js
const add = (a) => (b) => (c) => a + b + c;
add(1)(2)(3); // 6
```

### Why use it

- Pre-configure functions: `const add10 = add(10); add10(5) → 15`
- Compose small unary functions
- Functional APIs (Ramda, lodash/fp)

### Generic `curry()` implementation

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...next) => curried(...args, ...next);
  };
}

function sum(a, b, c) {
  return a + b + c;
}

const curriedSum = curry(sum);
curriedSum(1)(2)(3);    // 6
curriedSum(1, 2)(3);    // 6
curriedSum(1, 2, 3);    // 6
```

### Infinite currying (add until empty call)

```js
function add(a) {
  return function (b) {
    if (b === undefined) return a;
    return add(a + b);
  };
}

add(1)(2)(3)(); // 6
```

---

## 2. Partial Application

### Definition

Fix **some** arguments upfront; rest supplied later. **Not** necessarily one-at-a-time.

```js
function multiply(a, b, c) {
  return a * b * c;
}

// Manual partial
const doubleFirst = (b, c) => multiply(2, b, c);
doubleFirst(3, 4); // 24

// bind is partial application
const triple = multiply.bind(null, 3);
triple(2, 4); // 24
```

### `partial` helper

```js
function partial(fn, ...fixed) {
  return (...rest) => fn(...fixed, ...rest);
}

const greet = (greeting, name, punctuation) =>
  `${greeting}, ${name}${punctuation}`;

const sayHello = partial(greet, "Hello");
sayHello("World", "!"); // "Hello, World!"
```

### React use case — event handlers

```js
// Partial: fix first arg (id)
const onSelect = (id) => () => setSelected(id);

items.map((item) => (
  <button key={item.id} onClick={onSelect(item.id)}>
    {item.name}
  </button>
));
```

---

## 3. Currying vs Partial Application

| | Currying | Partial application |
|---|----------|---------------------|
| Args | One at a time | Any number fixed |
| Arity | Transforms function shape | Same final arity |
| Example | `f(a)(b)(c)` | `f(a, b, c)` → fix `a` |
| Tooling | `curry()` | `bind`, `partial()` |

**Both use closures** to remember fixed values.

```js
// Curried
const add = (a) => (b) => a + b;
const add5 = add(5);

// Partial
const addPartial = partial((a, b) => a + b, 5);
// add5(3) === addPartial(3) === 8
```

---

## 4. Advanced Patterns

### Uncurry (reverse)

```js
function uncurry(fn) {
  return function (...args) {
    let result = fn;
    for (const arg of args) result = result(arg);
    return result;
  };
}
```

### Placeholder partial (lodash style)

```js
const _ = Symbol("placeholder");
function partialRight(fn, ...fixed) {
  return (...rest) => {
    const args = fixed.map((a) => (a === _ ? rest.shift() : a));
    return fn(...args, ...rest);
  };
}
```

### Debounce as partial + closure

```js
function debounce(fn, ms) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), ms);
  };
}
// debounce(log, 300) — partial time, rest args at invoke
```

---

## 5. TypeScript

```ts
type Curried<A extends unknown[], R> = A extends [infer H, ...infer T]
  ? (arg: H) => Curried<T, R>
  : R;

// Practical: overload for known arity
function curry2<A, B, C>(fn: (a: A, b: B) => C): (a: A) => (b: B) => C {
  return (a) => (b) => fn(a, b);
}

const add = curry2((a: number, b: number) => a + b);
add(1)(2); // number
```

### Partial with generics

```ts
function partial<T extends unknown[], U>(
  fn: (...args: T) => U,
  ...head: Partial<T>
): (...tail: T) => U {
  return (...tail) => fn(...([...head, ...tail] as T));
}
```

---

## 6. Interview Quick Index

| Question | Section |
|----------|---------|
| What is currying? | [§1](#1-currying) |
| Implement `curry()` | [§1](#1-currying) |
| Partial vs currying | [§3](#3-currying-vs-partial-application) |
| `bind` as partial | [§2](#2-partial-application) |
| React onClick pattern | [§2](#2-partial-application) |

---

## Day 9 Cheat Sheet

```
Currying   → f(a,b,c) → f(a)(b)(c); unary chain
Partial    → fix some args now, rest later; bind()
Both       → closures hold fixed values
Interview  → implement curry(); explain bind vs curry
```

---

*End of Day 9 revision*
