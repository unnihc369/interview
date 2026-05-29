# Day 23 — setTimeout & setInterval

**Week 4 — Heaps / Timers** · **Topics:** Timers · Event loop · Debounce · Throttle · Timer IDs

---

## Table of Contents

1. [setTimeout Basics](#1-settimeout-basics)
2. [setInterval Basics](#2-setinterval-basics)
3. [Event Loop & Timer Queue](#3-event-loop--timer-queue)
4. [clearTimeout & clearInterval](#4-cleartimeout--clearinterval)
5. [Debounce & Throttle](#5-debounce--throttle)
6. [Heap-Scheduled Tasks Tie-In](#6-heap-scheduled-tasks-tie-in)
7. [React Patterns](#7-react-patterns)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. setTimeout Basics

### Syntax

```js
const id = setTimeout(callback, delayMs, ...args);
```

- Runs **once** after `delayMs` (minimum ~4ms in browsers after nesting depth).
- Returns numeric **timer ID** (Node: object with `ref`/`unref`).

```js
setTimeout(() => console.log("later"), 1000);
setTimeout(console.log, 1000, "hello"); // args passed to callback
```

### Zero delay

```js
setTimeout(() => console.log("timeout"), 0);
console.log("sync");
// sync → timeout (macrotask after current stack)
```

---

## 2. setInterval Basics

```js
const id = setInterval(() => {
  console.log("tick");
}, 1000);
```

Repeats every `delayMs` until `clearInterval(id)`.

### Pitfall — drift

Interval does not wait for callback to finish — can overlap. Use chained `setTimeout` for fixed gap **after** work:

```js
function schedule(fn, ms) {
  function tick() {
    fn();
    setTimeout(tick, ms);
  }
  setTimeout(tick, ms);
}
```

---

## 3. Event Loop & Timer Queue

Timers are **macrotasks** (task queue):

```
Call stack → microtasks (Promises) → macrotask (timer) → render
```

```js
Promise.resolve().then(() => console.log("micro"));
setTimeout(() => console.log("timer"), 0);
console.log("sync");
// sync, micro, timer
```

### Minimum delay clamping

HTML spec: nested timers (>5 deep) clamp to 4ms minimum.

---

## 4. clearTimeout & clearInterval

```js
const id = setTimeout(fn, 5000);
clearTimeout(id); // cancelled before fire

const ivate = setInterval(fn, 1000);
clearInterval(idate);
```

Always clear in React `useEffect` cleanup to prevent leaks.

---

## 5. Debounce & Throttle

### Debounce — wait for pause

```js
function debounce(fn, wait) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), wait);
  };
}

// Search input: API call 300ms after user stops typing
const search = debounce((q) => fetch(`/api?q=${q}`), 300);
```

### Throttle — at most once per window

```js
function throttle(fn, wait) {
  let last = 0;
  let timer;
  return function (...args) {
    const now = Date.now();
    const remaining = wait - (now - last);
    if (remaining <= 0) {
      last = now;
      fn.apply(this, args);
    } else if (!timer) {
      timer = setTimeout(() => {
        last = Date.now();
        timer = null;
        fn.apply(this, args);
      }, remaining);
    }
  };
}
```

| Pattern | Use |
|---------|-----|
| Debounce | Search, resize end, form validation |
| Throttle | Scroll, mousemove, rate-limit clicks |

---

## 6. Heap-Scheduled Tasks Tie-In

`setTimeout` internally uses timer heap in Node/libuv and browser timer structures — **earliest deadline first** (similar to min-heap).

For **K most frequent words chart** (Day 23 MC): batch DOM updates on interval; use heap for top-K word counts.

```js
function topKWords(text, k) {
  const freq = new Map();
  for (const w of text.toLowerCase().split(/\W+/)) {
    if (w) freq.set(w, (freq.get(w) ?? 0) + 1);
  }
  return [...freq.entries()]
    .sort((a, b) => b[1] - a[1] || a[0].localeCompare(b[0]))
    .slice(0, k);
}
```

---

## 7. React Patterns

```tsx
useEffect(() => {
  const id = setInterval(() => setCount((c) => c + 1), 1000);
  return () => clearInterval(id);
}, []);

useEffect(() => {
  const t = setTimeout(() => setDebouncedValue(value), 300);
  return () => clearTimeout(t);
}, [value]);
```

### Stale closure in timers

Use functional updates `setState(c => c+1)` or refs for latest values.

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| setTimeout vs setInterval | [§1](#1-settimeout-basics), [§2](#2-setinterval-basics) |
| Microtask vs macrotask | [§3](#3-event-loop--timer-queue) |
| Debounce vs throttle | [§5](#5-debounce--throttle) |
| Cleanup in React | [§7](#7-react-patterns) |

---

## Day 23 Cheat Sheet

```
setTimeout(fn, ms)  → once; macrotask
setInterval(fn, ms) → repeat; watch overlap drift
clearTimeout/Interval → always cleanup in useEffect
Debounce            → delay until quiet
Throttle            → max rate
Chained setTimeout  → fixed gap after async work
```

---

*End of Day 23 concepts*
