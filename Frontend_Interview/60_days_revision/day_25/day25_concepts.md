# Day 25 — Web Workers Basics

**Week 4 — Heaps / Concurrency** · **Topics:** Web Workers · postMessage · Transferables · Off-main-thread compute

---

## Table of Contents

1. [Why Web Workers](#1-why-web-workers)
2. [Dedicated Worker API](#2-dedicated-worker-api)
3. [postMessage & Structured Clone](#3-postmessage--structured-clone)
4. [Transferable Objects](#4-transferable-objects)
5. [Worker Patterns](#5-worker-patterns)
6. [Merge K Sorted in Worker](#6-merge-k-sorted-in-worker)
7. [Limitations & Alternatives](#7-limitations--alternatives)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. Why Web Workers

JavaScript on the main thread is **single-threaded**. Long CPU work blocks UI (jank, frozen inputs).

**Web Workers** run JS on **background threads** with no DOM access — ideal for:

| Task | Example |
|------|---------|
| Sorting / parsing large data | Merge K arrays |
| Image processing | Pixel filters |
| Crypto / hash | Client-side checksum |
| Heavy LeetCode-style compute | Matrix search |

---

## 2. Dedicated Worker API

### Main thread

```js
const worker = new Worker(new URL("./worker.js", import.meta.url), { type: "module" });

worker.postMessage({ type: "MERGE", arrays: [arr1, arr2] });

worker.onmessage = (e) => {
  console.log("Result:", e.data);
};

worker.onerror = (err) => console.error(err.message);

// Terminate when done
worker.terminate();
```

### worker.js

```js
self.onmessage = (e) => {
  const { type, arrays } = e.data;
  if (type === "MERGE") {
    const merged = mergeKSorted(arrays);
    self.postMessage({ merged });
  }
};

function mergeKSorted(arrays) {
  // min-heap merge — LC 23 style
  return arrays.flat().sort((a, b) => a - b);
}
```

### Vite / bundler note

Use `new URL(..., import.meta.url)` for worker entry resolution.

---

## 3. postMessage & Structured Clone

Messages are copied via **structured clone algorithm** (supports most objects, TypedArrays, Map, Set — not functions, DOM nodes).

```js
worker.postMessage({ nums: [1, 2, 3], meta: new Map([["k", 1]]) });
```

Large payloads copy is expensive — use **transferables**.

---

## 4. Transferable Objects

Transfer **ownership** of `ArrayBuffer` (zero-copy):

```js
const buffer = new Float32Array(1_000_000).buffer;
worker.postMessage({ buffer }, [buffer]);
// buffer detached on main thread — byteLength === 0
```

| Transferable | Use |
|--------------|-----|
| ArrayBuffer | Raw binary |
| MessagePort | Channel between workers |
| ImageBitmap | Canvas → worker |

---

## 5. Worker Patterns

### Request / response with id

```js
let reqId = 0;
const pending = new Map();

function callWorker(payload) {
  const id = ++reqId;
  return new Promise((resolve) => {
    pending.set(id, resolve);
    worker.postMessage({ id, ...payload });
  });
}

worker.onmessage = (e) => {
  const { id, result } = e.data;
  pending.get(id)?.(result);
  pending.delete(id);
};
```

### Worker pool

N workers + task queue for parallel chunks (image tiles, batch sort).

---

## 6. Merge K Sorted in Worker

```js
// worker: mergeK.js
function mergeKSorted(lists) {
  const heap = lists.map((arr, i) => ({ val: arr[0], i, j: 0 })).filter(x => x.val !== undefined);
  heap.sort((a, b) => a.val - b.val);
  const out = [];

  while (heap.length) {
    heap.sort((a, b) => a.val - b.val);
    const { val, i, j } = heap.shift();
    out.push(val);
    const next = lists[i][j + 1];
    if (next !== undefined) heap.push({ val: next, i, j: j + 1 });
  }
  return out;
}

self.onmessage = ({ data: lists }) => {
  self.postMessage(mergeKSorted(lists));
};
```

Main thread stays responsive while worker runs O(N log k).

---

## 7. Limitations & Alternatives

| Limitation | Workaround |
|------------|------------|
| No DOM | postMessage results; OffscreenCanvas |
| No shared state by default | SharedArrayBuffer + Atomics (Day 26–27) |
| Startup cost | Reuse worker pool |
| CORS worker scripts | Same-origin or blob URLs |

### Alternatives

- **requestIdleCallback** — low-priority main thread work
- **WASM** — near-native speed in worker
- **Service Worker** — network caching, not CPU

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| Worker vs async | [§1](#1-why-web-workers) — CPU vs I/O |
| Transferables | [§4](#4-transferable-objects) |
| Can worker touch DOM? | No |
| Module workers | `{ type: "module" }` |

---

## Day 25 Cheat Sheet

```
new Worker(url)     → background thread
postMessage(data)   → structured clone
transfer list       → zero-copy ArrayBuffer
onmessage           → receive result
terminate()         → kill worker
Use for             → CPU-heavy; not DOM
Merge K sorted      → heap in worker
```

---

*End of Day 25 concepts*
