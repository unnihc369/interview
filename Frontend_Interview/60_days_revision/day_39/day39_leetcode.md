# Day 39 — LeetCode (Kth & Successor)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 230 | [Kth Smallest Element in a BST](#1-kth-smallest-element-in-a-bst-230) | Medium | Inorder |
| 285 | [Inorder Successor in BST](#2-inorder-successor-in-bst-285) | Medium | BST search |
| 272 | [Closest Binary Search Tree Value](#3-closest-binary-search-tree-value-272) | Easy | BST walk |

---

## 1. Kth Smallest Element in a BST (230)

### JavaScript

```js
function kthSmallest(root, k) {
  const stack = [];
  let cur = root, count = 0;
  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop();
    if (++count === k) return cur.val;
    cur = cur.right;
  }
}
```

| Time | Space |
|------|-------|
| O(h + k) | O(h) |

---

## 2. Inorder Successor in BST (285)

### JavaScript

```js
function inorderSuccessor(root, p) {
  let succ = null, cur = root;
  while (cur) {
    if (p.val < cur.val) { succ = cur; cur = cur.left; }
    else cur = cur.right;
  }
  return succ;
}
```

| Time | Space |
|------|-------|
| O(h) | O(1) |

---

## 3. Closest Binary Search Tree Value (272)

### JavaScript

```js
function closestValue(root, target) {
  let cur = root, closest = cur.val;
  while (cur) {
    if (Math.abs(cur.val - target) < Math.abs(closest - target)) closest = cur.val;
    cur = target < cur.val ? cur.left : cur.right;
  }
  return closest;
}
```

| Time | Space |
|------|-------|
| O(h) | O(1) |

---

## Pattern Cheat Sheet

| Problem | Walk direction |
|---------|----------------|
| **230** | Inorder left-first |
| **285** | Go left when p.val < cur.val |
| **272** | Toward target, track closest |

---

*End of Day 39 LeetCode*
