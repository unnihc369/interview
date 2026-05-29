# Day 37 — BST Insert, Delete & Successor

**Topics:** Delete three cases · Inorder successor · Trim BST · Convert to greater tree

---

## Table of Contents

1. [Delete Node in BST](#1-delete-node-in-bst)
2. [Inorder Successor](#2-inorder-successor)
3. [Trim BST](#3-trim-bst)
4. [Convert BST to Greater Tree](#4-convert-bst-to-greater-tree)
5. [Interview Quick Index](#5-interview-quick-index)

---

## 1. Delete Node in BST

Three cases when deleting `key`:

| Case | Action |
|------|--------|
| **Leaf** | Remove node |
| **One child** | Replace with child |
| **Two children** | Replace with **inorder successor** (min of right subtree), delete successor |

```ts
function deleteNode(root: TreeNode | null, key: number): TreeNode | null {
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

---

## 2. Inorder Successor

**Successor of node x:** Smallest value **greater than x** → leftmost node in right subtree.

**Predecessor:** Largest in left subtree.

Used in delete (two-child case) and LC 285.

---

## 3. Trim BST

Remove nodes outside `[low, high]` range.

```ts
function trimBST(root: TreeNode | null, low: number, high: number): TreeNode | null {
  if (!root) return null;
  if (root.val < low) return trimBST(root.right, low, high);
  if (root.val > high) return trimBST(root.left, low, high);
  root.left = trimBST(root.left, low, high);
  root.right = trimBST(root.right, low, high);
  return root;
}
```

---

## 4. Convert BST to Greater Tree

Reverse inorder: right → root → left, accumulate sum.

```ts
function convertBST(root: TreeNode | null): TreeNode | null {
  let sum = 0;
  function dfs(node: TreeNode | null) {
    if (!node) return;
    dfs(node.right);
    sum += node.val;
    node.val = sum;
    dfs(node.left);
  }
  dfs(root);
  return root;
}
```

Every node becomes original value + sum of all greater values.

---

## 5. Interview Quick Index

| Question | Answer |
|----------|--------|
| Delete leaf? | Return null / child |
| Delete two children? | Copy successor val, delete successor |
| Successor? | Min of right subtree |
| Trim? | Skip subtrees entirely out of range |
| Greater tree? | Reverse inorder accumulation |

---

## Day 37 Cheat Sheet

```
Delete  → 0/1 child: replace; 2 children: successor swap
Successor → leftmost of right subtree
Trim    → prune left if too small, right if too big
Greater → reverse inorder running sum
```

---

*End of Day 37 concepts*
