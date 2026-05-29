# Day 36 — BST Property & Search

**Topics:** BST invariant · Search O(h) · Insert · Validate overview · Frontend index analogies

---

## Table of Contents

1. [BST Definition](#1-bst-definition)
2. [Search in BST](#2-search-in-bst)
3. [Insert into BST](#3-insert-into-bst)
4. [Validate BST Preview](#4-validate-bst-preview)
5. [BST vs Hash Map](#5-bst-vs-hash-map)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. BST Definition

For every node:
- All values in **left subtree** < node.val
- All values in **right subtree** > node.val
- **No duplicates** (LeetCode default) or define left ≤ / right >

```ts
class TreeNode {
  val: number;
  left: TreeNode | null = null;
  right: TreeNode | null = null;
  constructor(val: number) { this.val = val; }
}
```

**Inorder traversal → strictly increasing sequence** (if valid BST).

---

## 2. Search in BST

```ts
function searchBST(root: TreeNode | null, val: number): TreeNode | null {
  let cur = root;
  while (cur) {
    if (val === cur.val) return cur;
    cur = val < cur.val ? cur.left : cur.right;
  }
  return null;
}
```

| Average | Worst (skewed) |
|---------|----------------|
| O(log n) | O(n) |

Same as binary search on sorted array — but dynamic structure.

---

## 3. Insert into BST

```ts
function insertIntoBST(root: TreeNode | null, val: number): TreeNode {
  if (!root) return new TreeNode(val);
  if (val < root.val) root.left = insertIntoBST(root.left, val);
  else root.right = insertIntoBST(root.right, val);
  return root;
}
```

**Iterative insert** avoids stack overflow on deep trees.

---

## 4. Validate BST Preview

Wrong check: only compare immediate children.

```ts
// WRONG
node.left.val < node.val && node.right.val > node.val
```

Correct: entire left subtree < node < entire right subtree.

```ts
function isValidBST(root: TreeNode | null, min = -Infinity, max = Infinity): boolean {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

Full treatment Day 38.

---

## 5. BST vs Hash Map

| | BST | HashMap |
|--|-----|---------|
| Search | O(log n) avg | O(1) avg |
| Sorted order | Free via inorder | Need sort |
| Range queries | Efficient | Poor |
| Frontend use | Autocomplete trie/BST index | Object lookup |

**Dictionary BST index:** prefix search, ordered keys for typeahead.

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| BST property? | left < root < right (all descendants) |
| Search time? | O(h) |
| Inorder of BST? | Sorted ascending |
| Validate pitfall? | Must pass min/max bounds |
| Insert where? | Leaf position by comparison |

---

## Day 36 Cheat Sheet

```
BST     → left < node < right (entire subtrees)
Search  → while loop compare, go left/right
Insert  → recurse to null, attach new node
Validate → min/max bounds, not just children
```

---

*End of Day 36 concepts*
