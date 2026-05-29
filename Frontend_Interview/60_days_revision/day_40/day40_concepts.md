# Day 40 — BST to Sorted Array

**Topics:** Inorder → sorted array · Sorted array → balanced BST · Linked list to BST · Mode in BST

---

## Table of Contents

1. [Inorder Export](#1-inorder-export)
2. [Sorted Array to Balanced BST](#2-sorted-array-to-balanced-bst)
3. [Sorted Linked List to BST](#3-sorted-linked-list-to-bst)
4. [Find Mode in BST](#4-find-mode-in-bst)
5. [Frontend Export Use Cases](#5-frontend-export-use-cases)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Inorder Export

```ts
function bstToSortedArray(root: TreeNode | null): number[] {
  const result: number[] = [];
  function inorder(node: TreeNode | null) {
    if (!node) return;
    inorder(node.left);
    result.push(node.val);
    inorder(node.right);
  }
  inorder(root);
  return result;
}
```

O(n) time — natural **CSV/JSON export** of ordered products or keys.

---

## 2. Sorted Array to Balanced BST

Pick **middle** element as root → balanced height O(log n).

```ts
function sortedArrayToBST(nums: number[]): TreeNode | null {
  function build(lo: number, hi: number): TreeNode | null {
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

**Why mid?** Equal left/right subtree sizes → minimum height.

---

## 3. Sorted Linked List to BST

LC 109: Use slow/fast to find mid, or convert to array first (simpler in interviews).

```ts
function sortedListToBST(head: ListNode | null): TreeNode | null {
  const nums: number[] = [];
  while (head) { nums.push(head.val); head = head.next; }
  return sortedArrayToBST(nums);
}
```

In-place O(n) uses inorder simulation with list pointer (advanced).

---

## 4. Find Mode in BST

LC 501: Inorder scan — track run length of equal consecutive values.

```ts
function findMode(root: TreeNode | null): number[] {
  let maxCount = 0, count = 0, prev = NaN;
  const modes: number[] = [];

  function inorder(node: TreeNode | null) {
    if (!node) return;
    inorder(node.left);
    count = node.val === prev ? count + 1 : 1;
    if (count > maxCount) { maxCount = count; modes.length = 0; modes.push(node.val); }
    else if (count === maxCount) modes.push(node.val);
    prev = node.val;
    inorder(node.right);
  }
  inorder(root);
  return modes;
}
```

---

## 5. Frontend Export Use Cases

| Feature | Technique |
|---------|-----------|
| Export sorted products | Inorder → JSON/CSV download |
| Rebuild index after bulk import | Sort array → balanced BST |
| Restore from API sorted list | `sortedArrayToBST` |
| Most frequent price tag | Mode via inorder |

```ts
function exportSortedCSV(root: TreeNode | null): string {
  return bstToSortedArray(root).join("\n");
}
```

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| BST to sorted? | Inorder traversal |
| Sorted to balanced BST? | Mid element recursive split |
| Time sortedArrayToBST? | O(n) |
| Mode in BST? | Inorder run counting |
| Why balance? | Avoid skew O(n) operations |

---

## Day 40 Cheat Sheet

```
Export   → inorder → sorted array
Import   → mid-index recursive build
Balance  → always pick middle of range
Mode     → inorder consecutive duplicates
```

---

*End of Day 40 concepts*
