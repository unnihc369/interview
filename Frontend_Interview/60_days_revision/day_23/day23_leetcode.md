# Day 23 — LeetCode (Heaps / Greedy)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 692 | [Top K Frequent Words](#1-top-k-frequent-words-692) | Medium | Min-heap + tie-break |
| 973 | [K Closest Points to Origin](#2-k-closest-points-to-origin-973) | Medium | Max-heap size K |
| 787 | [Cheapest Flights Within K Stops](#3-cheapest-flights-within-k-stops-787) | Medium | Dijkstra / heap BFS |

---

## Table of Contents

1. [Top K Frequent Words (692)](#1-top-k-frequent-words-692)
2. [K Closest Points to Origin (973)](#2-k-closest-points-to-origin-973)
3. [Cheapest Flights Within K Stops (787)](#3-cheapest-flights-within-k-stops-787)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Top K Frequent Words (692)

### Problem (short)

Return **k** most frequent words; sort by freq desc, then **lexicographic asc** for ties.

```
Input: words = ["i","love","leetcode","i","love","coding"], k = 2
Output: ["i","love"]
```

### Hint 1 — Count with Map

### Hint 2 — Min-heap size k with custom compare

Pop worse element when size > k.

### Solution — JavaScript

```js
/**
 * @param {string[]} words
 * @param {number} k
 * @return {string[]}
 */
function topKFrequent(words, k) {
  const freq = new Map();
  for (const w of words) freq.set(w, (freq.get(w) ?? 0) + 1);

  return [...freq.entries()]
    .sort((a, b) => {
      if (b[1] !== a[1]) return b[1] - a[1];
      return a[0].localeCompare(b[0]);
    })
    .slice(0, k)
    .map(([w]) => w);
}
```

| Time | Space |
|------|-------|
| O(n log n) sort; O(n log k) heap | O(n) |

---

## 2. K Closest Points to Origin (973)

### Problem (short)

Return **k** points closest to origin `(0,0)`.

```
Input: points = [[1,3],[-2,2]], k = 1
Output: [[-2,2]]
```

### Hint 1 — Distance squared avoids sqrt

`x² + y²` — same ordering as distance.

### Hint 2 — Max-heap of size k

Keep k smallest; if new point closer than heap max, replace.

### Solution — JavaScript

```js
/**
 * @param {number[][]} points
 * @param {number} k
 * @return {number[][]}
 */
function kClosest(points, k) {
  const dist = ([x, y]) => x * x + y * y;
  return points
    .map((p) => ({ p, d: dist(p) }))
    .sort((a, b) => a.d - b.d)
    .slice(0, k)
    .map(({ p }) => p);
}
```

### Quickselect O(n) average — mention in interview

| Time | Space |
|------|-------|
| O(n log k) heap; O(n log n) sort | O(k) |

---

## 3. Cheapest Flights Within K Stops (787)

### Problem (short)

Directed weighted flights; cheapest price from `src` to `dst` with **at most k stops**.

```
Input: n = 3, flights = [[0,1,100],[1,2,100],[0,2,500]], src=0, dst=2, k=1
Output: 200
```

### Hint 1 — Bellman-Ford K+1 relaxations

Simple O(k * E).

### Hint 2 — Modified Dijkstra with (node, stops) state

Min-heap on cost; skip if stops > k.

### Solution — JavaScript (Bellman-Ford style)

```js
/**
 * @param {number} n
 * @param {number[][]} flights
 * @param {number} src
 * @param {number} dst
 * @param {number} k
 * @return {number}
 */
function findCheapestPrice(n, flights, src, dst, k) {
  let prices = Array(n).fill(Infinity);
  prices[src] = 0;

  for (let i = 0; i <= k; i++) {
    const tmp = [...prices];
    for (const [from, to, cost] of flights) {
      if (prices[from] === Infinity) continue;
      tmp[to] = Math.min(tmp[to], prices[from] + cost);
    }
    prices = tmp;
  }
  return prices[dst] === Infinity ? -1 : prices[dst];
}
```

| Time | Space |
|------|-------|
| O(k * E) | O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | Structure | Key |
|---------|-----------|-----|
| **692** | Map + heap/sort | Freq desc, word asc |
| **973** | Max-heap size k | Distance squared |
| **787** | k-relax / heap BFS | Stops budget |

### Dijkstra heap version (787)

```js
function findCheapestPriceDijkstra(n, flights, src, dst, k) {
  const adj = Array.from({ length: n }, () => []);
  for (const [f, t, p] of flights) adj[f].push([t, p]);
  const heap = [[0, src, 0]];
  const best = new Map();

  while (heap.length) {
    heap.sort((a, b) => a[0] - b[0]);
    const [cost, u, stops] = heap.shift();
    if (u === dst) return cost;
    if (stops > k) continue;
    const key = `${u},${stops}`;
    if (best.has(key) && best.get(key) <= cost) continue;
    best.set(key, cost);
    for (const [v, p] of adj[u]) heap.push([cost + p, v, stops + 1]);
  }
  return -1;
}
```

### Follow-ups

| Question | Answer |
|----------|--------|
| 692 tie-break | Lexicographic asc when freq equal |
| 973 avoid sqrt | Compare x²+y² |
| 787 k stops meaning | At most k intermediate stops |

---

*End of Day 23 LeetCode*
