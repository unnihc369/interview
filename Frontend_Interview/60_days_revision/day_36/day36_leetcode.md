# Day 36 — LeetCode (BST Search & Insert)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 700 | [Search in a Binary Search Tree](#1-search-in-a-binary-search-tree-700) | Easy | BST walk |
| 701 | [Insert into a Binary Search Tree](#2-insert-into-a-binary-search-tree-701) | Medium | BST insert |
| 98 | [Validate Binary Search Tree](#3-validate-binary-search-tree-98) | Medium | Min/max bounds |

---

## 1. Search in a Binary Search Tree (700)

### JavaScript

```js
function searchBST(root, val) {
  while (root) {
    if (val === root.val) return root;
    root = val < root.val ? root.left : root.right;
  }
  return null;
}
```

| Time | Space |
|------|-------|
| O(h) | O(1) |

---

## 2. Insert into a Binary Search Tree (701)

### JavaScript — Recursive

```js
function insertIntoBST(root, val) {
  if (!root) return new TreeNode(val);
  if (val < root.val) root.left = insertIntoBST(root.left, val);
  else root.right = insertIntoBST(root.right, val);
  return root;
}
```

### JavaScript — Iterative

```js
function insertIntoBST(root, val) {
  if (!root) return new TreeNode(val);
  let cur = root;
  while (true) {
    if (val < cur.val) {
      if (!cur.left) { cur.left = new TreeNode(val); break; }
      cur = cur.left;
    } else {
      if (!cur.right) { cur.right = new TreeNode(val); break; }
      cur = cur.right;
    }
  }
  return root;
}
```

| Time | Space |
|------|-------|
| O(h) | O(h) recursive / O(1) iterative |

---

## 3. Validate Binary Search Tree (98)

### JavaScript — Bounds

```js
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

### JavaScript — Inorder

```js
function isValidBST(root) {
  let prev = -Infinity;
  function inorder(node) {
    if (!node) return true;
    if (!inorder(node.left)) return false;
    if (node.val <= prev) return false;
    prev = node.val;
    return inorder(node.right);
  }
  return inorder(root);
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## Pattern Cheat Sheet

| Problem | Key |
|---------|-----|
| **700** | Compare and walk |
| **701** | Insert at leaf position |
| **98** | Bounds or strictly increasing inorder |

---

*End of Day 36 LeetCode*
