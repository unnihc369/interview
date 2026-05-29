# Day 41 — LeetCode (Balance & Leaves)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 110 | [Balanced Binary Tree](#1-balanced-binary-tree-110) | Easy | Height DFS |
| 1382 | [Balance a Binary Search Tree](#2-balance-a-binary-search-tree-1382) | Medium | Inorder + rebuild |
| 1302 | [Deepest Leaves Sum](#3-deepest-leaves-sum-1302) | Medium | BFS last level |

---

## 1. Balanced Binary Tree (110)

### JavaScript

```js
function isBalanced(root) {
  function height(node) {
    if (!node) return 0;
    const L = height(node.left);
    if (L === -1) return -1;
    const R = height(node.right);
    if (R === -1) return -1;
    if (Math.abs(L - R) > 1) return -1;
    return 1 + Math.max(L, R);
  }
  return height(root) !== -1;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 2. Balance a Binary Search Tree (1382)

### JavaScript

```js
function balanceBST(root) {
  const nums = [];
  (function inorder(n) {
    if (!n) return;
    inorder(n.left);
    nums.push(n.val);
    inorder(n.right);
  })(root);

  function build(lo, hi) {
    if (lo > hi) return null;
    const mid = (lo + hi) >> 1;
    const node = new TreeNode(nums[mid]);
    node.left = build(lo, mid - 1);
    node.right = build(mid + 1, hi);
    return node;
  }
  return build(0, nums.length - 1);
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## 3. Deepest Leaves Sum (1302)

### JavaScript — BFS

```js
function deepestLeavesSum(root) {
  let queue = [root], sum = root.val;
  while (queue.length) {
    const next = [], levelSum = 0;
    for (const node of queue) {
      if (node.left) { next.push(node.left); levelSum += node.left.val; }
      if (node.right) { next.push(node.right); levelSum += node.right.val; }
    }
    if (next.length) { queue = next; sum = levelSum; }
  }
  return sum;
}
```

| Time | Space |
|------|-------|
| O(n) | O(w) |

---

## Pattern Cheat Sheet

| Problem | Approach |
|---------|----------|
| **110** | Height with -1 sentinel |
| **1382** | Inorder + mid rebuild |
| **1302** | BFS, track last level sum |

---

*End of Day 41 LeetCode*
