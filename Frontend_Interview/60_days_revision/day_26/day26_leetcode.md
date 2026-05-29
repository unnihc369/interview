# Day 26 — LeetCode (Heaps / Greedy)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 621 | [Task Scheduler](#1-task-scheduler-621) | Medium | Max-heap + cooldown |
| 767 | [Reorganize String](#2-reorganize-string-767) | Medium | Max-heap greedy |
| 358 | [Rearrange String k Distance Apart](#3-rearrange-string-k-distance-apart-358) | Hard | Heap + queue |

---

## Table of Contents

1. [Task Scheduler (621)](#1-task-scheduler-621)
2. [Reorganize String (767)](#2-reorganize-string-767)
3. [Rearrange String k Distance Apart (358)](#3-rearrange-string-k-distance-apart-358)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Task Scheduler (621)

### Problem (short)

Tasks labeled A-Z; same task needs **n** cooldown slots. Minimum intervals to complete all (with idle slots).

```
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8  → A B idle A B idle A B
```

### Hint 1 — Max frequency drives answer

Formula: `(maxFreq - 1) * (n + 1) + countMaxFreq` vs `tasks.length`.

### Hint 2 — Max-heap simulation

Always schedule most frequent available task; cooldown queue.

### Solution — JavaScript (formula)

```js
/**
 * @param {character[]} tasks
 * @param {number} n
 * @return {number}
 */
function leastInterval(tasks, n) {
  const freq = new Map();
  for (const t of tasks) freq.set(t, (freq.get(t) ?? 0) + 1);
  const counts = [...freq.values()];
  const maxFreq = Math.max(...counts);
  const maxCount = counts.filter((c) => c === maxFreq).length;
  return Math.max(tasks.length, (maxFreq - 1) * (n + 1) + maxCount);
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) alphabet |

---

## 2. Reorganize String (767)

### Problem (short)

Rearrange string so no two adjacent chars same. Return any valid string or "".

```
Input: s = "aab"
Output: "aba"
```

### Hint 1 — Max-heap on char freq

Pop two most frequent, append, push back with decremented count.

### Solution — JavaScript

```js
/**
 * @param {string} s
 * @return {string}
 */
function reorganizeString(s) {
  const freq = new Map();
  for (const c of s) freq.set(c, (freq.get(c) ?? 0) + 1);

  const heap = [...freq.entries()].sort((a, b) => b[1] - a[1]);
  let res = "";

  while (heap.length) {
    heap.sort((a, b) => b[1] - a[1]);
    const [c1, f1] = heap.shift();
    if (!res.length || res[res.length - 1] !== c1) {
      res += c1;
      if (f1 > 1) heap.push([c1, f1 - 1]);
    } else {
      if (!heap.length) return "";
      heap.sort((a, b) => b[1] - a[1]);
      const [c2, f2] = heap.shift();
      res += c2;
      if (f2 > 1) heap.push([c2, f2 - 1]);
      heap.push([c1, f1]);
    }
  }
  return res.length === s.length ? res : "";
}
```

| Time | Space |
|------|-------|
| O(n log 26) | O(26) |

---

## 3. Rearrange String k Distance Apart (358)

### Problem (short)

Rearrange so **same character** appears at least **k** positions apart.

### Hint 1 — Max-heap + cooldown queue (like 621)

After using char, put in queue until k slots passed.

### Solution sketch — JavaScript

```js
function rearrangeString(s, k) {
  if (k === 0) return s;
  const freq = new Map();
  for (const c of s) freq.set(c, (freq.get(c) ?? 0) + 1);

  const heap = [...freq.entries()].sort((a, b) => b[1] - a[1]);
  const cooldown = [];
  let res = "";

  while (heap.length || cooldown.length) {
    if (cooldown.length && cooldown[0][1] === res.length) {
      heap.push(cooldown.shift());
      heap.sort((a, b) => b[1] - a[1]);
    }
    if (!heap.length) return "";
    heap.sort((a, b) => b[1] - a[1]);
    const [c, f] = heap.shift();
    res += c;
    if (f > 1) cooldown.push([c, f - 1, res.length + k - 1]);
  }
  return res.length === s.length ? res : "";
}
```

| Time | Space |
|------|-------|
| O(n log 26) | O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | Heap | Key |
|---------|------|-----|
| **621** | Max freq | Cooldown formula or sim |
| **767** | Max char freq | No adjacent equal |
| **358** | Max + queue | k-distance cooldown |

### Simulation code for 621 (heap)

```js
function leastIntervalSim(tasks, n) {
  const freq = new Map();
  for (const t of tasks) freq.set(t, (freq.get(t) ?? 0) + 1);
  const heap = [...freq.values()].sort((a, b) => b - a);
  let time = 0;
  const queue = [];

  while (heap.length || queue.length) {
    time++;
    if (heap.length) {
      const f = heap.shift() - 1;
      if (f > 0) queue.push([f, time + n]);
    }
    if (queue.length && queue[0][1] === time) {
      heap.push(queue.shift()[0]);
      heap.sort((a, b) => b - a);
    }
  }
  return time;
}
```

### Follow-ups

| Question | Answer |
|----------|--------|
| 621 idle slots | Formula handles idle explicitly |
| 767 impossible case | Most frequent > (n+1)/2 → "" |
| 358 k=1 | Reduces to 767 |

---

*End of Day 26 LeetCode*
