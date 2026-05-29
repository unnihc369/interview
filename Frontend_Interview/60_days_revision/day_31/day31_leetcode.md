# Day 31 — LeetCode (BFS Level Order)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 102 | [Binary Tree Level Order Traversal](#1-binary-tree-level-order-traversal-102) | Medium | BFS |
| 107 | [Binary Tree Level Order Traversal II](#2-binary-tree-level-order-traversal-ii-107) | Medium | BFS + reverse |
| 199 | [Binary Tree Right Side View](#3-binary-tree-right-side-view-199) | Medium | BFS last per level |

---

## 1. Binary Tree Level Order Traversal (102)

### Problem

Return values grouped **level by level**.

### JavaScript

```js
function levelOrder(root) {
  if (!root) return [];
  const res = [], queue = [root];
  while (queue.length) {
    const size = queue.length, level = [];
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    res.push(level);
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n) | O(w) |

---

## 2. Binary Tree Level Order Traversal II (107)

### Problem

Same as 102 but return levels **bottom-up**.

### JavaScript

```js
function levelOrderBottom(root) {
  const topDown = levelOrder(root);
  return topDown.reverse();
}
```

Or push each level to front: `res.unshift(level)`.

| Time | Space |
|------|-------|
| O(n) | O(w) |

---

## 3. Binary Tree Right Side View (199)

### Problem

Return values visible from the **right side** (rightmost node per level).

### Hint

Last node processed in each BFS level batch.

### JavaScript — BFS

```js
function rightSideView(root) {
  if (!root) return [];
  const res = [], queue = [root];
  while (queue.length) {
    const size = queue.length;
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      if (i === size - 1) res.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  return res;
}
```

### JavaScript — DFS (right first)

```js
function rightSideViewDFS(root) {
  const res = [];
  function dfs(node, depth) {
    if (!node) return;
    if (depth === res.length) res.push(node.val);
    dfs(node.right, depth + 1);
    dfs(node.left, depth + 1);
  }
  dfs(root, 0);
  return res;
}
```

| Time | Space |
|------|-------|
| O(n) | O(w) BFS / O(h) DFS |

---

## Pattern Cheat Sheet

| Problem | Key |
|---------|-----|
| **102** | BFS + level batch |
| **107** | Reverse result or unshift |
| **199** | Last in level OR DFS right-first |

---

*End of Day 31 LeetCode*
