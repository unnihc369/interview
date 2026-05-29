# Day 18 — Generator Functions

**Week 3 — Two Pointers** · **Topics:** `function*` · `yield` · `yield*` · Lazy sequences · Infinite iterators

---

## Table of Contents

1. [Generator Syntax](#1-generator-syntax)
2. [yield & Iterator Protocol](#2-yield--iterator-protocol)
3. [Lazy Evaluation & Memory](#3-lazy-evaluation--memory)
4. [yield* Delegation](#4-yield-delegation)
5. [Generator Methods: next, return, throw](#5-generator-methods-next-return-throw)
6. [Practical Patterns](#6-practical-patterns)
7. [Two-Pointer Generators](#7-two-pointer-generators)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. Generator Syntax

### Basic generator

```js
function* countUpTo(n) {
  for (let i = 1; i <= n; i++) {
    yield i;
  }
}

const gen = countUpTo(3);
gen.next(); // { value: 1, done: false }
gen.next(); // { value: 2, done: false }
gen.next(); // { value: 3, done: false }
gen.next(); // { value: undefined, done: true }
```

Generators **pause** at each `yield` and resume on next `.next()`.

### Iterable by default

```js
for (const x of countUpTo(5)) console.log(x);
[...countUpTo(4)]; // [1,2,3,4]
```

### Arrow functions cannot be generators

No `* =>` syntax — use `function*` or generator method in object/class.

```js
const obj = {
  *range(n) {
    for (let i = 0; i < n; i++) yield i;
  },
};
```

---

## 2. yield & Iterator Protocol

Calling a generator returns a **Generator object** implementing both **Iterator** and **Iterable** (`[Symbol.iterator]` returns `this`).

```js
function* ids() {
  let id = 0;
  while (true) yield ++id;
}

const g = ids();
g.next().value; // 1
g.next().value; // 2
```

### Passing values into generator

```js
function* echo() {
  let x = yield "ready";
  yield `got: ${x}`;
}

const g = echo();
g.next();      // { value: "ready", done: false }
g.next(42);    // { value: "got: 42", done: false }
```

The argument to `next(val)` becomes the **result** of the previous `yield` expression.

---

## 3. Lazy Evaluation & Memory

Generators produce values **on demand** — O(1) memory for infinite streams vs materializing arrays.

```js
function* readLines(chunks) {
  for (const chunk of chunks) {
    for (const line of chunk.split("\n")) {
      if (line) yield line;
    }
  }
}
```

### vs Array methods

| Approach | Memory | When |
|----------|--------|------|
| `[...Array(1e6).keys()]` | O(n) | Need random access |
| `function* range(n)` | O(1) | Stream / pipeline |

### Pipeline pattern

```js
function* map(iterable, fn) {
  for (const x of iterable) yield fn(x);
}

function* filter(iterable, pred) {
  for (const x of iterable) if (pred(x)) yield x;
}

const pipeline = filter(map(countUpTo(10), (x) => x * 2), (x) => x > 5);
[...pipeline]; // [6, 8, 10, 12, 14, 16, 18, 20]
```

---

## 4. yield* Delegation

`yield*` delegates iteration to another iterable/generator.

```js
function* a() {
  yield 1;
  yield 2;
}

function* b() {
  yield* a();
  yield 3;
}

[...b()]; // [1, 2, 3]
```

### Flatten nested arrays

```js
function* flatten(arr) {
  for (const item of arr) {
    if (Array.isArray(item)) yield* flatten(item);
    else yield item;
  }
}
```

---

## 5. Generator Methods: next, return, throw

```js
function* gen() {
  try {
    yield 1;
    yield 2;
  } catch (e) {
    yield `caught: ${e.message}`;
  } finally {
    console.log("cleanup");
  }
}

const g = gen();
g.next();           // { value: 1, done: false }
g.throw(new Error("fail")); // caught → { value: "caught: fail", done: false }
g.return(99);       // { value: 99, done: true } — triggers finally
```

Useful for **cancelable** async flows and testing cleanup.

---

## 6. Practical Patterns

### Unique ID factory

```js
function* createIdGenerator(prefix = "id") {
  let n = 0;
  while (true) yield `${prefix}-${++n}`;
}

const nextId = createIdGenerator("todo");
nextId.next().value; // "todo-1"
```

### Paginated API fetcher

```js
async function* fetchPages(url, pageSize = 20) {
  let page = 1;
  while (true) {
    const res = await fetch(`${url}?page=${page}&size=${pageSize}`);
    const data = await res.json();
    if (!data.items.length) return;
    yield data.items;
    page++;
  }
}
```

### Onboarding steps (Day 18 MC)

```js
function* onboardingSteps() {
  yield { slide: 1, title: "Welcome" };
  yield { slide: 2, title: "Features" };
  yield { slide: 3, title: "Permissions" };
  yield { slide: 4, title: "Done" };
}
```

---

## 7. Two-Pointer Generators

Yield pairs from opposite ends (palindrome pairs, merge walk):

```js
function* endPairs(arr) {
  let l = 0, r = arr.length - 1;
  while (l <= r) {
    yield [l, r, arr[l], arr[r]];
    l++;
    r--;
  }
}

// Dutch flag step tracer for LC 75
function* dutchFlagSteps(nums) {
  let lo = 0, mid = 0, hi = nums.length - 1;
  while (mid <= hi) {
    yield { nums: [...nums], lo, mid, hi };
    if (nums[mid] === 0) {
      [nums[lo], nums[mid]] = [nums[mid], nums[lo]];
      lo++; mid++;
    } else if (nums[mid] === 1) {
      mid++;
    } else {
      [nums[mid], nums[hi]] = [nums[hi], nums[mid]];
      hi--;
    }
  }
}
```

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| Generator vs normal function | [§1](#1-generator-syntax) — pauses at yield |
| Lazy vs eager | [§3](#3-lazy-evaluation--memory) |
| `yield*` purpose | [§4](#4-yield-delegation) |
| Pass value to generator | [§2](#2-yield--iterator-protocol) — `next(arg)` |
| Infinite loop safe? | Yes if consumed partially |

---

## Day 18 Cheat Sheet

```
function*  → returns Generator (iterable + iterator)
yield v    → pause, emit v
yield* it  → delegate to iterable
next(x)    → x becomes previous yield's result
Lazy       → O(1) memory for streams
for...of   → works on generators natively
```

---

*End of Day 18 concepts*
