# Day 42 — BST Week Revision

**Topics:** BST property · Search/insert/delete · Validate · Kth · Sorted conversion · Balance · Range sum

---

## Table of Contents

1. [Week 6 Concept Map](#1-week-6-concept-map)
2. [BST Operation Summary](#2-bst-operation-summary)
3. [Pattern Templates](#3-pattern-templates)
4. [Complexity Reference](#4-complexity-reference)
5. [Common Mistakes](#5-common-mistakes)
6. [Mock Interview Prompts](#6-mock-interview-prompts)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Week 6 Concept Map

| Day | Core topic | Key LC |
|-----|------------|--------|
| 36 | Search & insert | 700, 701, 98 |
| 37 | Delete & transform | 450, 669, 538 |
| 38 | Validate & iterator | 98, 99, 173 |
| 39 | Kth & successor | 230, 285, 272 |
| 40 | Sorted ↔ BST | 109, 108, 501 |
| 41 | Balance | 110, 1382, 1302 |

---

## 2. BST Operation Summary

| Operation | Approach | Time |
|-----------|----------|------|
| Search | Walk left/right | O(h) |
| Insert | Leaf by comparison | O(h) |
| Delete | 0/1/2 child cases | O(h) |
| Validate | Min/max bounds | O(n) |
| Kth smallest | Inorder count | O(h+k) |
| To sorted | Inorder | O(n) |
| From sorted | Mid split build | O(n) |
| Balance | Inorder + rebuild | O(n) |

---

## 3. Pattern Templates

### BST search

```ts
while (cur) {
  if (val === cur.val) return cur;
  cur = val < cur.val ? cur.left : cur.right;
}
```

### Validate

```ts
function valid(node, min, max): boolean {
  if (!node) return true;
  if (node.val <= min || node.val >= max) return false;
  return valid(node.left, min, node.val) && valid(node.right, node.val, max);
}
```

### Kth smallest

```ts
// inorder iterative, increment count until k
```

### Range sum (bonus — LC 938)

```ts
function rangeSumBST(root, low, high) {
  if (!root) return 0;
  let sum = (root.val >= low && root.val <= high) ? root.val : 0;
  if (root.val > low) sum += rangeSumBST(root.left, low, high);
  if (root.val < high) sum += rangeSumBST(root.right, low, high);
  return sum;
}
```

---

## 4. Complexity Reference

| | Balanced | Skewed |
|--|----------|--------|
| Search/insert/delete | O(log n) | O(n) |
| Space | O(h) = O(log n) | O(n) |

Always mention **skewed worst case** in interviews.

---

## 5. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Validate only direct children | Pass min/max bounds |
| Delete 2-child: wrong successor | Use **leftmost of right** subtree |
| Kth largest confusion | Reverse inorder (right first) |
| `<=` vs `<` on bounds | Match problem duplicate policy |
| Unbalanced after insert | Rebuild or use AVL (conceptual) |

---

## 6. Mock Interview Prompts

1. Live BST visualizer with insert/search/delete.
2. Validate nested form like BST bounds.
3. Export inorder sorted CSV from product BST.
4. **Timed LC mock:** 98, 230, 99 (45 min).

---

## 7. Interview Quick Index

| Need | Pattern |
|------|---------|
| Sorted output | Inorder |
| Balanced build | Mid of sorted array |
| Kth element | Inorder / reverse inorder |
| Fix swapped nodes | Two inorder violations |
| Range query | BST prune + accumulate |

---

## Day 42 Cheat Sheet

```
BST week → search, insert, delete, validate, kth, balance
Inorder  → sorted order & kth & mode
Validate → min/max bounds everywhere
Mock     → 98 + 230 + 99 in 45 min
```

---

*End of Day 42 concepts*
