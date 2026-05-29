# Day 55 — LeetCode (Tree DP)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 337 | [House Robber III](#1-house-robber-iii-337) | Medium | Tree DP rob/skip |
| 124 | [Binary Tree Maximum Path Sum](#2-binary-tree-maximum-path-sum-124) | Hard | Review — global max |
| 968 | [Binary Tree Cameras](#3-binary-tree-cameras-968) | Hard | Multi-state tree DP |

---

## Table of Contents

1. [House Robber III (337)](#1-house-robber-iii-337)
2. [Binary Tree Maximum Path Sum (124)](#2-binary-tree-maximum-path-sum-124)
3. [Binary Tree Cameras (968)](#3-binary-tree-cameras-968)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. House Robber III (337)

### Solution

```js
function rob(root) {
  function dfs(node) {
    if (!node) return [0, 0];
    const [lr, ls] = dfs(node.left);
    const [rr, rs] = dfs(node.right);
    return [node.val + ls + rs, Math.max(lr, ls) + Math.max(rr, rs)];
  }
  const [a, b] = dfs(root);
  return Math.max(a, b);
}
```

---

## 2. Binary Tree Maximum Path Sum (124)

### Solution (Review)

```js
function maxPathSum(root) {
  let ans = -Infinity;
  function gain(node) {
    if (!node) return 0;
    const L = Math.max(0, gain(node.left));
    const R = Math.max(0, gain(node.right));
    ans = Math.max(ans, node.val + L + R);
    return node.val + Math.max(L, R);
  }
  gain(root);
  return ans;
}
```

---

## 3. Binary Tree Cameras (968)

### Solution

```js
function minCameraCover(root) {
  let cams = 0;
  function dfs(node) {
    if (!node) return 2;
    let state = 0;
    for (const child of [node.left, node.right]) {
      const s = dfs(child);
      if (s === 0) state = 0;
      if (s === 2 && state !== 0) state = 1;
    }
    if (state === 0) { cams++; return 2; }
    return state === 1 ? 1 : 0;
  }
  return (dfs(root) === 0 ? 1 : 0) + cams;
}
```

---

## 4. Pattern Cheat Sheet

| Problem | States | Order |
|---------|--------|-------|
| 337 | rob / skip | post-order |
| 124 | max gain upward | post-order + global |
| 968 | uncovered / covered / camera | post-order greedy |

---

**Prev/Next:** [Concepts](day55_concepts.md) · [Machine Coding](day55_machine_coding.md)
