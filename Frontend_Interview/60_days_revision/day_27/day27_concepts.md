# Day 27 — Atomics

**Week 4 — Heaps / Concurrency** · **Topics:** Atomics API · wait/notify · Lock-free counters · Thread-safe shared state

---

## Table of Contents

1. [Why Atomics](#1-why-atomics)
2. [Atomics Methods](#2-atomics-methods)
3. [Compare-Exchange & Lock-Free](#3-compare-exchange--lock-free)
4. [Atomics.wait & Atomics.notify](#4-atomicswait--atomicsnotify)
5. [Schedule Posts by Timestamp Heap](#5-schedule-posts-by-timestamp-heap)
6. [Offline Priority Sync](#6-offline-priority-sync)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Why Atomics

With **SharedArrayBuffer**, multiple threads read/write same memory → **data races**.

```js
// UNSAFE — lost updates possible
view[0] = view[0] + 1;
```

**Atomics** guarantee indivisible read-modify-write on TypedArray elements backed by SAB.

---

## 2. Atomics Methods

| Method | Effect |
|--------|--------|
| `Atomics.load(arr, i)` | Read |
| `Atomics.store(arr, i, v)` | Write |
| `Atomics.add(arr, i, delta)` | Add and return old value |
| `Atomics.sub(arr, i, delta)` | Subtract |
| `Atomics.exchange(arr, i, v)` | Swap and return old |
| `Atomics.compareExchange(arr, i, expected, replacement)` | CAS |

```js
const sab = new SharedArrayBuffer(4);
const counter = new Int32Array(sab);

Atomics.store(counter, 0, 0);
Atomics.add(counter, 0, 1); // returns 0, counter now 1
Atomics.load(counter, 0);   // 1
```

Works only on **Int8Array, Uint8Array, Int16Array, Uint16Array, Int32Array, Uint32Array, BigInt64Array, BigUint64Array** over SAB.

---

## 3. Compare-Exchange & Lock-Free

### CAS pattern (spinlock sketch)

```js
function tryLock(lock, expected = 0, value = 1) {
  return Atomics.compareExchange(lock, 0, expected, value) === expected;
}

function unlock(lock) {
  Atomics.store(lock, 0, 0);
}
```

### Safe increment

```js
function incrementShared(counter, index) {
  Atomics.add(counter, index, 1);
}
```

---

## 4. Atomics.wait & Atomics.notify

Block worker until index changes (worker only for `wait`):

```js
// Worker
while (Atomics.load(flag, 0) === 0) {
  Atomics.wait(flag, 0, 0); // sleep until notify
}
// proceed

// Main
Atomics.store(flag, 0, 1);
Atomics.notify(flag, 0, 1); // wake one waiter
```

Use for **producer/consumer** without busy polling.

---

## 5. Schedule Posts by Timestamp Heap

**Day 27 MC:** Posts `{ id, content, publishAt }` in min-heap by timestamp. Worker checks heap head vs `Date.now()` using shared clock counter.

```js
class ScheduledPostHeap {
  constructor() {
    this.data = [];
  }
  push(post) {
    this.data.push(post);
    this.data.sort((a, b) => a.publishAt - b.publishAt);
  }
  peek() { return this.data[0]; }
  pop() { return this.data.shift(); }
}

// Shared: Int32 publish tick
const tick = new Int32Array(new SharedArrayBuffer(4));
// Main Atomics.store(tick, 0, Date.now()) periodically
```

---

## 6. Offline Priority Sync

**RN offline sync:** Queue mutations in IndexedDB/SQLite with priority. On reconnect, pop min-heap priority queue; use Atomics on shared `syncInProgress` flag to prevent double sync from worker + main.

```js
const syncLock = new Int32Array(new SharedArrayBuffer(4));
if (Atomics.compareExchange(syncLock, 0, 0, 1) !== 0) return; // already syncing
try {
  await flushQueue();
} finally {
  Atomics.store(syncLock, 0, 0);
  Atomics.notify(syncLock, 0);
}
```

---

## 7. Interview Quick Index

| Question | Section |
|----------|---------|
| Atomics vs mutex | Lock-free CAS; lower overhead |
| Which arrays? | Int32 etc. on SAB only |
| wait/notify use | [§4](#4-atomicswait--atomicsnotify) |
| Lost update problem | [§1](#1-why-atomics) |

---

## Day 27 Cheat Sheet

```
Atomics.load/store     → safe read/write
Atomics.add            → atomic increment
compareExchange        → CAS lock-free
wait/notify            → worker sleep/wake
Requires               → SharedArrayBuffer + isolation
Schedule posts         → min-heap by publishAt
```

---

*End of Day 27 concepts*
