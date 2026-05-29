# Day 10 — JavaScript & TypeScript Revision

**Topics:** Proxy · Reflect · Metaprogramming · Validation patterns

---

## Table of Contents

1. [Proxy Fundamentals](#1-proxy-fundamentals)
2. [Common Proxy Traps](#2-common-proxy-traps)
3. [Reflect API](#3-reflect-api)
4. [Real-World Patterns](#4-real-world-patterns)
5. [Interview Traps](#5-interview-traps)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Proxy Fundamentals

### Definition

**Proxy** wraps an object and intercepts operations (get, set, delete, etc.) via **traps**.

```js
const user = { name: "Ada", age: 30 };

const proxy = new Proxy(user, {
  get(target, prop, receiver) {
    console.log(`GET ${String(prop)}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    console.log(`SET ${String(prop)} = ${value}`);
    return Reflect.set(target, prop, value, receiver);
  },
});

proxy.name;       // logs GET name → "Ada"
proxy.age = 31;   // logs SET age = 31
```

### Why use Proxy

| Use | Example |
|-----|---------|
| Validation | Reject invalid assignments |
| Logging / devtools | Trace access |
| Reactive state | Vue 3 reactivity (conceptually) |
| Default values | Return 0 for missing numeric keys |
| Immutability | Block sets/deletes |

---

## 2. Common Proxy Traps

| Trap | Intercepts |
|------|------------|
| `get` | Property read |
| `set` | Property write |
| `has` | `in` operator |
| `deleteProperty` | `delete obj.k` |
| `ownKeys` | `Object.keys` |
| `apply` | Function call (proxy of function) |
| `construct` | `new` (proxy of constructor) |

### Validated user object

```js
function createValidatedUser(initial) {
  return new Proxy(initial, {
    set(target, prop, value) {
      if (prop === "age" && (typeof value !== "number" || value < 0)) {
        throw new TypeError("age must be non-negative number");
      }
      if (prop === "email" && !String(value).includes("@")) {
        throw new TypeError("invalid email");
      }
      target[prop] = value;
      return true;
    },
  });
}
```

### Negative array index (Python style)

```js
function createArray(...items) {
  return new Proxy(items, {
    get(target, prop) {
      const i = Number(prop);
      if (Number.isInteger(i) && i < 0) {
        return target[target.length + i];
      }
      return Reflect.get(target, prop);
    },
  });
}

const arr = createArray(10, 20, 30);
arr[-1]; // 30
```

### Readonly proxy

```js
function readonly(obj) {
  return new Proxy(obj, {
    set() {
      return false;
    },
    deleteProperty() {
      return false;
    },
  });
}
```

---

## 3. Reflect API

### Purpose

- **Default behavior** for traps (forward to target correctly)
- Fixes inconsistencies (`Reflect.set` return boolean vs `obj.x = y`)
- Works with Proxies — `Reflect` methods are **trap invokers**

```js
const obj = { a: 1 };

Reflect.get(obj, "a");           // 1
Reflect.set(obj, "b", 2);        // true
Reflect.has(obj, "a");           // true
Reflect.deleteProperty(obj, "a"); // true
Reflect.ownKeys(obj);            // ["b"]
```

### Reflect vs Object

| Operation | Prefer |
|-----------|--------|
| `Reflect.get/set` | Inside Proxy traps |
| `Object.keys` | External introspection |
| `in` / `delete` | User code; traps use Reflect |

### Reflect.construct

```js
function Person(name) {
  this.name = name;
}
const p = Reflect.construct(Person, ["Ada"]);
```

---

## 4. Real-World Patterns

### Observable store (mini reactive)

```js
function observable(obj, onChange) {
  return new Proxy(obj, {
    set(target, prop, value) {
      const prev = target[prop];
      const ok = Reflect.set(target, prop, value);
      if (prev !== value) onChange(prop, value, prev);
      return ok;
    },
  });
}

const state = observable({ count: 0 }, (k, v) => console.log(k, v));
state.count = 1;
```

### Default value proxy

```js
const withDefaults = (target, defaults) =>
  new Proxy(target, {
    get(t, p, r) {
      if (p in t) return Reflect.get(t, p, r);
      if (p in defaults) return defaults[p];
      return undefined;
    },
  });
```

### Function memoization via Proxy (apply trap)

```js
function memoizeFn(fn) {
  const cache = new Map();
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      const key = JSON.stringify(args);
      if (cache.has(key)) return cache.get(key);
      const result = Reflect.apply(target, thisArg, args);
      cache.set(key, result);
      return result;
    },
  });
}
```

---

## 5. Interview Traps

| Trap | Detail |
|------|--------|
| Revocable proxy | `Proxy.revocable()` — later access throws |
| Proxy on primitives | Must wrap object box first |
| Performance | Proxy adds overhead — not for hot loops |
| Invariants | Some ops can't be fully overridden (non-configurable props) |
| JSON.stringify | May bypass get trap for own enumerable props |

```js
const { proxy, revoke } = Proxy.revocable({ x: 1 }, {});
revoke();
// proxy.x → TypeError
```

---

## 6. Interview Quick Index

| Question | Section |
|----------|---------|
| What is Proxy? | [§1](#1-proxy-fundamentals) |
| Common traps | [§2](#2-common-proxy-traps) |
| Why Reflect? | [§3](#3-reflect-api) |
| Validation with Proxy | [§2](#2-common-proxy-traps) |
| Vue reactivity link | [§4](#4-real-world-patterns) |

---

## Day 10 Cheat Sheet

```
Proxy   → wrap object, intercept ops via traps
Reflect → default forwarding; use inside traps
Pairs   → get/set/has/deleteProperty/ownKeys
Use     → validation, logging, reactive state, defaults
```

---

*End of Day 10 revision*
