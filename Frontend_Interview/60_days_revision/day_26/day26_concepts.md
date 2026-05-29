# Day 26 — SharedArrayBuffer

**Week 4 — Heaps / Concurrency** · **Topics:** SharedArrayBuffer · COOP/COEP · Shared memory · vs postMessage

---

## Table of Contents

1. [SharedArrayBuffer Overview](#1-sharedarraybuffer-overview)
2. [Security Requirements (COOP/COEP)](#2-security-requirements-coopcoep)
3. [Creating & Viewing Shared Memory](#3-creating--viewing-shared-memory)
4. [Workers + Shared Memory](#4-workers--shared-memory)
5. [Top-K Dashboard Pattern](#5-top-k-dashboard-pattern)
6. [When NOT to Use SAB](#6-when-not-to-use-sab)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. SharedArrayBuffer Overview

### Problem with postMessage

Structured clone **copies** large buffers — expensive for real-time data (audio, metrics, game state).

**SharedArrayBuffer (SAB)** allocates memory **shared** between main thread and workers — same bytes visible to all.

```js
const sab = new SharedArrayBuffer(1024); // 1 KB shared
const view = new Int32Array(sab);
view[0] = 42; // visible to all threads with same SAB
```

### Spectre mitigation

Browsers disabled SAB briefly; re-enabled with **cross-origin isolation** headers.

---

## 2. Security Requirements (COOP/COEP)

Page must send:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

Check:

```js
self.crossOriginIsolated; // true when headers set
typeof SharedArrayBuffer !== "undefined";
```

Without isolation, `SharedArrayBuffer` may be undefined in browsers.

---

## 3. Creating & Viewing Shared Memory

```js
const BYTES = 4 * 1000; // 1000 Int32 slots
const sab = new SharedArrayBuffer(BYTES);
const metrics = new Int32Array(sab);

// Main thread writes
metrics[0] = 1; // request count

// Worker reads same memory
// worker.js: const metrics = new Int32Array(sab);
```

Multiple TypedArray views on one SAB:

```js
const u8 = new Uint8Array(sab);
const i32 = new Int32Array(sab); // same underlying bytes
```

---

## 4. Workers + Shared Memory

### Main

```js
const sab = new SharedArrayBuffer(4);
const worker = new Worker("worker.js");
worker.postMessage({ sab }); // shared, NOT transferred
```

### Worker

```js
self.onmessage = (e) => {
  const view = new Int32Array(e.data.sab);
  view[0] = 99;
  self.postMessage("done");
};
```

**Race conditions** without Atomics — use Atomics (Day 27) for safe read-modify-write.

---

## 5. Top-K Dashboard Pattern

**Top K expensive products** (Day 26 MC): worker computes heap top-K from shared metric buffer; main thread renders dashboard reading shared counts.

```js
// Shared layout: [version, count0, count1, ...]
const sab = new SharedArrayBuffer(4 * (1 + MAX_PRODUCTS));
const data = new Int32Array(sab);

function workerLoop() {
  while (true) {
    // recompute top K from data slice
    const topK = computeTopK(data.slice(1));
    Atomics.store(data, 0, Atomics.load(data, 0) + 1); // bump version
    postTopK(topK);
  }
}
```

Main polls version or uses `Atomics.wait` (Day 27).

---

## 6. When NOT to Use SAB

| Prefer postMessage | Prefer SAB |
|--------------------|------------|
| Small messages | High-frequency large numeric data |
| Simple request/response | Ring buffers, audio samples |
| No COOP/COEP control | crossOriginIsolated app |

### RN note

SharedArrayBuffer less common in React Native — use JSI / native shared memory for perf-critical paths.

---

## 7. Interview Quick Index

| Question | Section |
|----------|---------|
| SAB vs transfer | [§1](#1-sharedarraybuffer-overview) — shared vs move |
| Why COOP/COEP? | [§2](#2-security-requirements-coopcoep) — Spectre |
| Thread-safe updates? | Atomics (Day 27) |
| crossOriginIsolated? | [§2](#2-security-requirements-coopcoep) |

---

## Day 26 Cheat Sheet

```
SharedArrayBuffer  → same memory in workers + main
postMessage(sab)   → share reference (no transfer list)
COOP + COEP        → required for SAB in browser
TypedArray view    → read/write shared bytes
Races              → fix with Atomics
Top-K dashboard    → worker writes shared metrics heap
```

---

*End of Day 26 concepts*
