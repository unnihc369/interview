# Day 13 — JavaScript & TypeScript Revision (Saturday)

**Topics:** WeakMap · WeakSet · Garbage collection · Private metadata

---

## Table of Contents

1. [WeakMap](#1-weakmap)
2. [WeakSet](#2-weakset)
3. [Weak vs Strong Collections](#3-weak-vs-strong-collections)
4. [Use Cases](#4-use-cases)
5. [Interview Quick Index](#5-interview-quick-index)

---

## 1. WeakMap

### Definition

Keys must be **objects** (or registered symbols). Keys are held **weakly** — don't prevent GC.

```js
const wm = new WeakMap();
const obj = { id: 1 };
wm.set(obj, "metadata");
wm.get(obj); // "metadata"
```

| Feature | WeakMap |
|---------|---------|
| Keys | Objects only |
| Iterable | No |
| `size` | No |
| GC | Key collected → entry removed |

### Private data pattern

```js
const privateData = new WeakMap();

class User {
  constructor(name) {
    privateData.set(this, { name, token: crypto.randomUUID() });
  }
  getName() {
    return privateData.get(this).name;
  }
}
```

---

## 2. WeakSet

### Definition

Stores **objects only**, weakly held. No duplicates.

```js
const visited = new WeakSet();
const node = document.querySelector("div");
visited.add(node);
visited.has(node); // true
```

Use: mark objects processed without leaking memory (DOM nodes, graph traversal).

---

## 3. Weak vs Strong Collections

| | Map / Set | WeakMap / WeakSet |
|---|-----------|-------------------|
| Keys/values | Any | Objects only |
| GC | Strong refs block collection | Weak refs |
| Iteration | Yes | No |
| Use when | General cache | Metadata tied to object lifetime |

### Memory leak with Map

```js
const cache = new Map();
function handle(el) {
  cache.set(el, computeHeavy(el)); // el can't GC while in map
}
```

### WeakMap fix

```js
const cache = new WeakMap();
function handle(el) {
  cache.set(el, computeHeavy(el)); // when el removed from DOM, entry gone
}
```

---

## 4. Use Cases

| Pattern | Collection |
|---------|------------|
| DOM node metadata | WeakMap |
| Instance private fields | WeakMap (or `#field`) |
| Cycle-safe marking | WeakSet |
| Memo by object identity | WeakMap |
| React fiber / internal slots | Conceptually similar |

### WeakMap + sliding window chart (bridge to machine coding)

Store per-canvas-element chart state without global leak:

```js
const chartState = new WeakMap();

function initChart(canvasEl) {
  chartState.set(canvasEl, { points: [], windowSize: 50 });
}

function addPoint(canvasEl, value) {
  const state = chartState.get(canvasEl);
  state.points.push(value);
  if (state.points.length > state.windowSize) state.points.shift();
}
```

---

## 5. Interview Quick Index

| Question | Section |
|----------|---------|
| WeakMap vs Map | [§3](#3-weak-vs-strong-collections) |
| Why no iteration? | [§1](#1-weakmap) — GC timing |
| Private data | [§1, §4](#4-use-cases) |
| DOM metadata | [§4](#4-use-cases) |

---

## Day 13 Cheat Sheet

```
WeakMap  → object keys, weak refs, no size/iterate
WeakSet  → unique objects, weak membership
Use      → metadata, private data, no memory leaks
Avoid    → primitive keys, needing cache size
```

---

*End of Day 13 revision*
