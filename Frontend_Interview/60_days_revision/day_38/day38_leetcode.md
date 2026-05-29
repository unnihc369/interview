# Day 38 — LeetCode (Validate & Iterator)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 98 | [Validate Binary Search Tree](#1-validate-binary-search-tree-98) | Medium | Bounds / inorder |
| 99 | [Recover Binary Search Tree](#2-recover-binary-search-tree-99) | Medium | Inorder violations |
| 173 | [Binary Search Tree Iterator](#3-binary-search-tree-iterator-173) | Medium | Stack iterator |

---

## 1. Validate Binary Search Tree (98)

### JavaScript — Bounds

```js
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 2. Recover Binary Search Tree (99)

### JavaScript

```js
function recoverTree(root) {
  let first = null, second = null, prev = null;
  const stack = [];
  let cur = root;

  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop();
    if (prev && cur.val < prev.val) {
      if (!first) first = prev;
      second = cur;
    }
    prev = cur;
    cur = cur.right;
  }
  [first.val, second.val] = [second.val, first.val];
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 3. Binary Search Tree Iterator (173)

### JavaScript

```js
class BSTIterator {
  constructor(root) {
    this.stack = [];
    this._pushLeft(root);
  }
  _pushLeft(node) {
    while (node) { this.stack.push(node); node = node.left; }
  }
  next() {
    const node = this.stack.pop();
    this._pushLeft(node.right);
    return node.val;
  }
  hasNext() {
    return this.stack.length > 0;
  }
}
```

| Operation | Amortized |
|-----------|-----------|
| `next()` | O(1) |
| `hasNext()` | O(1) |

---

## Pattern Cheat Sheet

| Problem | Core idea |
|---------|-----------|
| **98** | Pass (min, max) down |
| **99** | Two inorder inversions |
| **173** | Lazy inorder with stack |

---

*End of Day 38 LeetCode*
