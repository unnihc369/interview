# Day 40 — LeetCode (Sorted ↔ BST)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 109 | [Convert Sorted List to Binary Search Tree](#1-convert-sorted-list-to-binary-search-tree-109) | Medium | Mid split |
| 108 | [Convert Sorted Array to Binary Search Tree](#2-convert-sorted-array-to-binary-search-tree-108) | Easy | Mid split |
| 501 | [Find Mode in Binary Search Tree](#3-find-mode-in-binary-search-tree-501) | Easy | Inorder runs |

---

## 1. Convert Sorted List to Binary Search Tree (109)

### JavaScript (array helper — interview acceptable)

```js
function sortedListToBST(head) {
  const nums = [];
  while (head) { nums.push(head.val); head = head.next; }
  return sortedArrayToBST(nums);
}

function sortedArrayToBST(nums) {
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
| O(n) | O(n) array / O(log n) stack |

---

## 2. Convert Sorted Array to Binary Search Tree (108)

### JavaScript

```js
function sortedArrayToBST(nums) {
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
| O(n) | O(log n) |

---

## 3. Find Mode in Binary Search Tree (501)

### JavaScript

```js
function findMode(root) {
  let maxCount = 0, count = 0, prev = null;
  const modes = [];

  (function inorder(node) {
    if (!node) return;
    inorder(node.left);
    count = node.val === prev ? count + 1 : 1;
    if (count > maxCount) { maxCount = count; modes.length = 0; modes.push(node.val); }
    else if (count === maxCount) modes.push(node.val);
    prev = node.val;
    inorder(node.right);
  })(root);

  return modes;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## Pattern Cheat Sheet

| Problem | Technique |
|---------|-----------|
| **108/109** | Mid index → balanced BST |
| **501** | Inorder duplicate runs |

---

*End of Day 40 LeetCode*
