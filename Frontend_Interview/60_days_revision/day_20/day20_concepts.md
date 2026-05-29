# Day 20 — Typed Arrays

**Week 3 — Two Pointers** · **Topics:** ArrayBuffer · TypedArray views · DataView · Binary data in frontend

---

## Table of Contents

1. [ArrayBuffer Basics](#1-arraybuffer-basics)
2. [TypedArray Family](#2-typedarray-family)
3. [DataView — Endian-Safe Reads](#3-dataview--endian-safe-reads)
4. [Performance & Memory Layout](#4-performance--memory-layout)
5. [Merge Sorted — Typed Array Version](#5-merge-sorted--typed-array-version)
6. [Web APIs Using Binary Data](#6-web-apis-using-binary-data)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. ArrayBuffer Basics

### Raw binary allocation

```js
const buffer = new ArrayBuffer(16); // 16 bytes
buffer.byteLength; // 16
```

`ArrayBuffer` is **fixed-length** raw memory — not directly readable. Use **views** (TypedArray / DataView).

### Slice & copy

```js
const copy = buffer.slice(0, 8); // copies bytes 0-7
```

### Transferable to Web Worker

```js
worker.postMessage(buffer, [buffer]); // ownership moves; main thread detached
```

---

## 2. TypedArray Family

All share same API shape: `length`, `[index]`, `set`, `subarray`, `byteOffset`, `buffer`.

| Type | Bytes | Range |
|------|-------|-------|
| Int8Array | 1 | -128..127 |
| Uint8Array | 1 | 0..255 |
| Int16Array | 2 | ... |
| Uint16Array | 2 | ... |
| Int32Array | 4 | ... |
| Uint32Array | 4 | ... |
| Float32Array | 4 | IEEE float |
| Float64Array | 8 | IEEE double |

```js
const buf = new ArrayBuffer(12);
const floats = new Float32Array(buf); // 3 elements
floats[0] = 1.5;
floats[1] = 2.5;
floats[2] = 3.5;
```

### From regular array

```js
const ta = new Int32Array([10, 20, 30]);
const fromArr = Int32Array.from([1, 2, 3], (x) => x * 2);
```

### Shared buffer, multiple views

```js
const buffer = new ArrayBuffer(4);
const u8 = new Uint8Array(buffer);
const i32 = new Int32Array(buffer);
u8[0] = 0xff;
// i32[0] now affected — same underlying bytes
```

---

## 3. DataView — Endian-Safe Reads

```js
const buffer = new ArrayBuffer(8);
const view = new DataView(buffer);

view.setInt16(0, 256, true);  // little-endian
view.getInt16(0, true);       // 256
view.getInt16(0, false);      // different if endian differs
```

Use when parsing **mixed-width** binary protocols (WebSocket binary frames, file headers).

---

## 4. Performance & Memory Layout

### Why TypedArrays?

| Benefit | Detail |
|---------|--------|
| Dense memory | No boxing like `Array` of numbers |
| SIMD-friendly | Canvas, WebGL, audio DSP |
| Zero-copy share | Workers + SharedArrayBuffer |

### vs regular Array

```js
// Regular — may be sparse, heterogeneous
const arr = [1.1, 2.2, 3.3];

// Typed — fixed type, contiguous
const f32 = new Float32Array([1.1, 2.2, 3.3]);
```

### Merge two sorted TypedArrays (LC 88 style)

```js
function mergeSortedTyped(a, b) {
  const out = new Int32Array(a.length + b.length);
  let i = 0, j = 0, k = 0;
  while (i < a.length && j < b.length) {
    out[k++] = a[i] <= b[j] ? a[i++] : b[j++];
  }
  while (i < a.length) out[k++] = a[i++];
  while (j < b.length) out[k++] = b[j++];
  return out;
}
```

---

## 5. Merge Sorted — Typed Array Version

Two-pointer merge is identical to LC 88 / 986 — only allocation differs.

```js
function mergeInPlace(nums1, m, nums2, n) {
  let p1 = m - 1, p2 = n - 1, write = m + n - 1;
  while (p2 >= 0) {
    if (p1 >= 0 && nums1[p1] > nums2[p2]) nums1[write--] = nums1[p1--];
    else nums1[write--] = nums2[p2--];
  }
}
```

Works on any indexable including `Int32Array`.

---

## 6. Web APIs Using Binary Data

| API | TypedArray use |
|-----|----------------|
| Canvas ImageData | `Uint8ClampedArray` RGBA |
| Web Audio | `Float32Array` samples |
| FileReader | `ArrayBuffer` → parse |
| fetch arrayBuffer | protobuf / wasm |
| WebGL buffers | `Float32Array` vertices |

### Canvas example

```js
const img = ctx.getImageData(0, 0, w, h);
const data = img.data; // Uint8ClampedArray
for (let i = 0; i < data.length; i += 4) {
  data[i] = 255 - data[i]; // invert R
}
ctx.putImageData(img, 0, 0);
```

---

## 7. Interview Quick Index

| Question | Section |
|----------|---------|
| ArrayBuffer vs TypedArray | [§1](#1-arraybuffer-basics), [§2](#2-typedarray-family) |
| Why not regular Array for audio? | [§4](#4-performance--memory-layout) |
| Endianness | [§3](#3-dataview--endian-safe-reads) |
| Transfer to worker | [§1](#1-arraybuffer-basics) |
| Merge sorted arrays | [§5](#5-merge-sorted--typed-array-version) |

---

## Day 20 Cheat Sheet

```
ArrayBuffer     → raw bytes; not directly indexable
TypedArray      → typed view on buffer; Int32, Float32, Uint8...
DataView        → read/write mixed sizes + endian
Merge sorted    → two pointers; same as LC 88
Worker transfer → postMessage(buf, [buf]) moves ownership
```

---

*End of Day 20 concepts*
