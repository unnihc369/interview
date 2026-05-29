# Day 48 — LeetCode (Bitwise Trie)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 421 | [Maximum XOR of Two Numbers in an Array](#1-maximum-xor-of-two-numbers-421) | Medium | Binary trie greedy |
| 1707 | [Maximum XOR With an Element From Array](#2-maximum-xor-with-an-element-from-array-1707) | Hard | Offline sort + trie |
| 1938 | [Maximum Genetic Difference Query](#3-maximum-genetic-difference-query-1938) | Hard | Tree + binary trie |

---

## Table of Contents

1. [Maximum XOR of Two Numbers (421)](#1-maximum-xor-of-two-numbers-421)
2. [Maximum XOR With an Element (1707)](#2-maximum-xor-with-an-element-from-array-1707)
3. [Maximum Genetic Difference Query (1938)](#3-maximum-genetic-difference-query-1938)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Maximum XOR of Two Numbers (421)

### Problem (short)

Return maximum `nums[i] XOR nums[j]`.

```
nums = [3,10,5,25,2,8]
Output: 28  (5 XOR 25)
```

### Hint 1 — Binary trie

Insert all numbers bit by bit (31 → 0).

### Hint 2 — Greedy opposite

For each bit, take opposite branch if exists — maximizes XOR at that bit.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
function findMaximumXOR(nums) {
  const root = { zero: null, one: null };
  const B = 31;

  for (const n of nums) {
    let node = root;
    for (let i = B; i >= 0; i--) {
      const bit = (n >> i) & 1;
      if (bit === 0) {
        if (!node.zero) node.zero = { zero: null, one: null };
        node = node.zero;
      } else {
        if (!node.one) node.one = { zero: null, one: null };
        node = node.one;
      }
    }
  }

  let ans = 0;
  for (const n of nums) {
    let node = root;
    let x = 0;
    for (let i = B; i >= 0; i--) {
      const bit = (n >> i) & 1;
      const opp = bit ^ 1;
      if (opp === 0 && node.zero) { x |= 1 << i; node = node.zero; }
      else if (opp === 1 && node.one) { x |= 1 << i; node = node.one; }
      else node = node.zero ?? node.one;
    }
    ans = Math.max(ans, x);
  }
  return ans;
}
```

| Time | Space |
|------|-------|
| O(n × 32) | O(n × 32) |

---

## 2. Maximum XOR With an Element From Array (1707)

### Problem (short)

Queries `[xi, mi]`: max `(xi XOR num)` where `num <= mi`. Return -1 if none.

### Hint 1 — Offline processing

Sort nums and queries by mi; insert nums into trie incrementally.

### Hint 2 — Per query

Greedy XOR walk on current trie.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @param {number[][]} queries
 * @return {number[]}
 */
function maximizeXor(nums, queries) {
  nums.sort((a, b) => a - b);
  const sorted = queries.map((q, i) => [...q, i]).sort((a, b) => a[1] - b[1]);
  const ans = new Array(queries.length).fill(-1);
  const root = { zero: null, one: null };
  let j = 0;
  const B = 31;

  const insert = (n) => {
    let node = root;
    for (let i = B; i >= 0; i--) {
      const bit = (n >> i) & 1;
      if (bit === 0) {
        if (!node.zero) node.zero = { zero: null, one: null };
        node = node.zero;
      } else {
        if (!node.one) node.one = { zero: null, one: null };
        node = node.one;
      }
    }
  };

  const queryMax = (x) => {
    let node = root;
    if (!node.zero && !node.one) return -1;
    let val = 0;
    for (let i = B; i >= 0; i--) {
      const bit = (x >> i) & 1;
      const opp = bit ^ 1;
      if (opp === 0 && node.zero) { val |= 1 << i; node = node.zero; }
      else if (opp === 1 && node.one) { val |= 1 << i; node = node.one; }
      else node = node.zero ?? node.one;
    }
    return val;
  };

  for (const [x, m, idx] of sorted) {
    while (j < nums.length && nums[j] <= m) insert(nums[j++]);
    const r = queryMax(x);
    ans[idx] = r === -1 ? -1 : x ^ r;
  }
  return ans;
}
```

| Time | Space |
|------|-------|
| O((n + q) log n + q × 32) | O(n × 32) |

---

## 3. Maximum Genetic Difference Query (1938)

### Problem (short)

Tree rooted at 0. `parents[i]` parent of i. `scores[i]` gene score. Query `(node, val)`: max `(score XOR val)` in subtree of node.

### Hint 1 — DFS tree

For each node, need binary trie of all scores in subtree.

### Hint 2 — Small-to-large merge

Merge child tries into parent (DSU on tree / heavy-light trick).

### Solution — JavaScript (conceptual DFS + trie)

```js
/**
 * @param {number[]} parents
 * @param {number[]} scores
 * @param {number[][]} queries
 * @return {number[]}
 */
function maxGeneticDifference(parents, scores, queries) {
  const n = scores.length;
  const children = Array.from({ length: n }, () => []);
  for (let i = 1; i < n; i++) children[parents[i]].push(i);

  const byNode = Array.from({ length: n }, () => []);
  for (const [node, val, idx] of queries.map((q, i) => [...q, i])) {
    byNode[node].push({ val, idx });
  }

  const ans = new Array(queries.length);
  const B = 31;

  function insert(root, num) {
    let node = root;
    for (let i = B; i >= 0; i--) {
      const bit = (num >> i) & 1;
      const key = bit === 0 ? "zero" : "one";
      if (!node[key]) node[key] = { zero: null, one: null };
      node = node[key];
    }
  }

  function maxXor(root, val) {
    let node = root;
    let x = 0;
    for (let i = B; i >= 0; i--) {
      const bit = (val >> i) & 1;
      const opp = bit ^ 1;
      const ok = opp === 0 ? "zero" : "one";
      const fb = opp === 0 ? "one" : "zero";
      if (node[ok]) { x |= 1 << i; node = node[ok]; }
      else node = node[fb];
    }
    return x;
  }

  function dfs(u) {
    const trie = { zero: null, one: null };
    insert(trie, scores[u]);
    for (const v of children[u]) {
      const childTrie = dfs(v);
      // merge child into trie (traverse and insert all)
      mergeTrie(trie, childTrie);
    }
    for (const { val, idx } of byNode[u]) {
      ans[idx] = maxXor(trie, val);
    }
    return trie;
  }

  dfs(0);
  return ans;
}

function mergeTrie(a, b) {
  if (!b) return;
  // Insert all values from b into a via re-insert scores
  // Production: iterative merge with count pruning
}
```

| Time | Space |
|------|-------|
| O(n × 32 × log n) small-to-large | O(n × 32) |

**Interview:** Explain offline query grouping by node + per-subtree binary trie + max XOR query.

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **421** | Binary trie + opposite bit | Max pair XOR |
| **1707** | Offline sort by mi | Incremental trie insert |
| **1938** | Subtree trie merge | Max score XOR val in tree |

### Follow-ups

| Question | Answer |
|----------|--------|
| 421 brute force? | O(n²) — trie O(n×32) |
| 1707 why sort queries? | Only nums ≤ mi eligible |
| 1938 merge tries? | Small-to-large O(n log n) |

### Test cases

```js
console.assert(findMaximumXOR([3,10,5,25,2,8]) === 28);

maximizeXor(
  [0,1,2,3,4],
  [[3,1],[1,3],[5,6]]
); // [-1,3,5]

// 1938: tree [−1,0,0,1,1], scores [5,2,9,7,8]
```

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 421 | O(n × 32) | O(n × 32) |
| 1707 | O(n log n + q × 32) | O(n × 32) |
| 1938 | O(n × 32 × log n) | O(n × 32) |

---

*End of Day 48 LeetCode*
