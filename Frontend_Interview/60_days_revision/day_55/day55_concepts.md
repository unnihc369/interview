# Day 55 — DP on Trees (House Robber)

**Week 8 — Dynamic Programming** · **Topics:** Tree DP · Post-order · Rob/not rob · Max path sum review

---

## Table of Contents

1. [Tree DP Pattern](#1-tree-dp-pattern)
2. [House Robber III](#2-house-robber-iii)
3. [Binary Tree Maximum Path Sum (Review)](#3-binary-tree-maximum-path-sum-review)
4. [Binary Tree Cameras](#4-binary-tree-cameras)
5. [General Tree DP Template](#5-general-tree-dp-template)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Tree DP Pattern

Process nodes **post-order**: children before parent.

Return tuple(s) representing best answers for subtrees.

```js
function dfs(node) {
  if (!node) return baseCase;
  const left = dfs(node.left);
  const right = dfs(node.right);
  // combine left + right + node.val
  return combinedState;
}
```

---

## 2. House Robber III

Each node has value. Cannot rob adjacent nodes (parent-child).

Return `[robThis, skipThis]` for subtree rooted at node.

```js
function rob(root) {
  function dfs(node) {
    if (!node) return [0, 0];
    const [lRob, lSkip] = dfs(node.left);
    const [rRob, rSkip] = dfs(node.right);
    const rob = node.val + lSkip + rSkip;
    const skip = Math.max(lRob, lSkip) + Math.max(rRob, rSkip);
    return [rob, skip];
  }
  const [robRoot, skipRoot] = dfs(root);
  return Math.max(robRoot, skipRoot);
}
```

---

## 3. Binary Tree Maximum Path Sum (Review)

Path can start/end anywhere. Global max updated when combining left + right through node.

```js
function maxPathSum(root) {
  let best = -Infinity;
  function gain(node) {
    if (!node) return 0;
    const left = Math.max(0, gain(node.left));
    const right = Math.max(0, gain(node.right));
    best = Math.max(best, node.val + left + right);
    return node.val + Math.max(left, right);
  }
  gain(root);
  return best;
}
```

---

## 4. Binary Tree Cameras

Min cameras to monitor all nodes. States: not covered / covered (no cam) / has camera.

```js
function minCameraCover(root) {
  let cameras = 0;
  // 0 = uncovered, 1 = covered no cam, 2 = has camera
  function dfs(node) {
    if (!node) return 1;
    const left = dfs(node.left);
    const right = dfs(node.right);
    if (left === 0 || right === 0) {
      cameras++;
      return 2;
    }
    if (left === 2 || right === 2) return 1;
    return 0;
  }
  return dfs(root) === 0 ? cameras + 1 : cameras;
}
```

---

## 5. General Tree DP Template

1. Identify state per node (include/exclude, covered states, max gain).
2. Post-order DFS returns state to parent.
3. Update global answer during combine step.
4. Handle null children explicitly.

---

## 6. Interview Quick Index

| LC | Return type | Global update |
|----|-------------|---------------|
| 337 | [rob, skip] | max at root |
| 124 | max gain to parent | best path through node |
| 968 | coverage state | camera count |

---

**Next:** [Machine Coding](day55_machine_coding.md) · [LeetCode](day55_leetcode.md)
