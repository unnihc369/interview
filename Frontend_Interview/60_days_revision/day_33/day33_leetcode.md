# Day 33 — LeetCode (LCA)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 236 | [Lowest Common Ancestor of a Binary Tree](#1-lowest-common-ancestor-of-a-binary-tree-236) | Medium | Post-order |
| 235 | [Lowest Common Ancestor of a BST](#2-lowest-common-ancestor-of-a-bst-235) | Medium | BST walk |
| 1644 | [Lowest Common Ancestor of a Binary Tree II](#3-lowest-common-ancestor-of-a-binary-tree-ii-1644) | Medium | Parent pointers |

---

## 1. Lowest Common Ancestor of a Binary Tree (236)

### JavaScript

```js
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;
  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);
  if (left && right) return root;
  return left || right;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 2. Lowest Common Ancestor of a BST (235)

### JavaScript

```js
function lowestCommonAncestor(root, p, q) {
  while (root) {
    if (p.val < root.val && q.val < root.val) root = root.left;
    else if (p.val > root.val && q.val > root.val) root = root.right;
    else return root;
  }
}
```

| Time | Space |
|------|-------|
| O(h) | O(1) |

---

## 3. Lowest Common Ancestor of a Binary Tree II (1644)

Nodes may not exist; p and q may have **parent** pointers.

### With parent pointers

```js
function lowestCommonAncestor(root, p, q) {
  const seen = new Set();
  let a = p;
  while (a) { seen.add(a); a = a.parent; }
  let b = q;
  while (b) {
    if (seen.has(b)) return b;
    b = b.parent;
  }
  return null;
}
```

### Without parent — verify existence first

Run 236 only if both nodes found during traversal; else return null.

| Time | Space |
|------|-------|
| O(h) | O(h) |

---

## Pattern Cheat Sheet

| Problem | Approach |
|---------|----------|
| **236** | Post-order split detection |
| **235** | BST ordering walk |
| **1644** | Ancestor set + walk up |

---

*End of Day 33 LeetCode*
