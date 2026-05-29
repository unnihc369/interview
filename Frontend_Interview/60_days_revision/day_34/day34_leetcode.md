# Day 34 — LeetCode (Advanced Tree)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 114 | [Flatten Binary Tree to Linked List](#1-flatten-binary-tree-to-linked-list-114) | Medium | Preorder / Morris |
| 543 | [Diameter of Binary Tree](#2-diameter-of-binary-tree-543) | Easy | DFS depth |
| 687 | [Longest Univ Value Path](#3-longest-univ-value-path-687) | Medium | DFS path |

---

## 1. Flatten Binary Tree to Linked List (114)

### Problem

Flatten to right-only linked list in **preorder**. Do in-place.

### JavaScript

```js
function flatten(root) {
  let cur = root;
  while (cur) {
    if (cur.left) {
      let pred = cur.left;
      while (pred.right) pred = pred.right;
      pred.right = cur.right;
      cur.right = cur.left;
      cur.left = null;
    }
    cur = cur.right;
  }
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 2. Diameter of Binary Tree (543)

### JavaScript

```js
function diameterOfBinaryTree(root) {
  let max = 0;
  function depth(node) {
    if (!node) return 0;
    const L = depth(node.left), R = depth(node.right);
    max = Math.max(max, L + R);
    return 1 + Math.max(L, R);
  }
  depth(root);
  return max;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 3. Longest Univ Value Path (687)

### JavaScript

```js
function longestUnivPath(root) {
  let max = 0;
  function dfs(node) {
    if (!node) return 0;
    let L = dfs(node.left), R = dfs(node.right);
    L = node.left?.val === node.val ? L + 1 : 0;
    R = node.right?.val === node.val ? R + 1 : 0;
    max = Math.max(max, L + R);
    return Math.max(L, R);
  }
  dfs(root);
  return max;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## Pattern Cheat Sheet

| Problem | Return vs global |
|---------|------------------|
| **114** | Mutate pointers in-place |
| **543** | Return depth; track diameter globally |
| **687** | Return longest same-val arm |

---

*End of Day 34 LeetCode*
