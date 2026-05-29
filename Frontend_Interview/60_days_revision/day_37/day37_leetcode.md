# Day 37 — LeetCode (BST Delete & Transform)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 450 | [Delete Node in a BST](#1-delete-node-in-a-bst-450) | Medium | Delete cases |
| 669 | [Trim a Binary Search Tree](#2-trim-a-binary-search-tree-669) | Medium | Prune |
| 538 | [Convert BST to Greater Tree](#3-convert-bst-to-greater-tree-538) | Medium | Reverse inorder |

---

## 1. Delete Node in a BST (450)

### JavaScript

```js
function deleteNode(root, key) {
  if (!root) return null;
  if (key < root.val) root.left = deleteNode(root.left, key);
  else if (key > root.val) root.right = deleteNode(root.right, key);
  else {
    if (!root.left) return root.right;
    if (!root.right) return root.left;
    let succ = root.right;
    while (succ.left) succ = succ.left;
    root.val = succ.val;
    root.right = deleteNode(root.right, succ.val);
  }
  return root;
}
```

| Time | Space |
|------|-------|
| O(h) | O(h) |

---

## 2. Trim a Binary Search Tree (669)

### JavaScript

```js
function trimBST(root, low, high) {
  if (!root) return null;
  if (root.val < low) return trimBST(root.right, low, high);
  if (root.val > high) return trimBST(root.left, low, high);
  root.left = trimBST(root.left, low, high);
  root.right = trimBST(root.right, low, high);
  return root;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 3. Convert BST to Greater Tree (538)

### JavaScript

```js
function convertBST(root) {
  let sum = 0;
  (function dfs(node) {
    if (!node) return;
    dfs(node.right);
    sum += node.val;
    node.val = sum;
    dfs(node.left);
  })(root);
  return root;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## Pattern Cheat Sheet

| Problem | Technique |
|---------|-----------|
| **450** | Successor for 2-child delete |
| **669** | Skip out-of-range branches |
| **538** | Reverse inorder prefix sum |

---

*End of Day 37 LeetCode*
