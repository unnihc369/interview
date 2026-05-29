# Day 42 — LeetCode (BST Mock Revision)

**Timed mock (45 min):** Solve without hints first, then verify.

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 98 | [Validate Binary Search Tree](#1-mock-98-validate-bst) | Medium | Min/max bounds |
| 230 | [Kth Smallest Element in a BST](#2-mock-230-kth-smallest) | Medium | Inorder |
| 99 | [Recover Binary Search Tree](#3-mock-99-recover-bst) | Medium | Inorder violations |

---

## 1. Mock — 98 Validate BST

### Self-check

- [ ] Used bounds, not just child comparison
- [ ] Handled `[-Infinity, Infinity]` initial range
- [ ] Strict inequality (`<=` / `>=` fails)

### Solution

```js
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

**Target time:** 12 min

---

## 2. Mock — 230 Kth Smallest

### Self-check

- [ ] Iterative inorder (no full array if optimizing)
- [ ] 1-indexed k
- [ ] Stopped early at k

### Solution

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

**Target time:** 15 min

---

## 3. Mock — 99 Recover BST

### Self-check

- [ ] Found two inorder violations
- [ ] `first` = first violation prev node
- [ ] `second` = latest violation current node
- [ ] Swapped values (not pointers)

### Solution

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

**Target time:** 18 min

---

## Mock Scoring Rubric

| Score | Criteria |
|-------|----------|
| **Pass** | 3/3 correct + state bounds vs inorder tradeoffs |
| **Partial** | 2/3 with clean iterative inorder |
| **Review** | Re-study Days 36–41 |

### Verbal follow-ups

1. Why is `[5, 1, 4, null, null, 3, 6]` invalid? (4 in left subtree of 5 but 4 > 1 — need bounds)
2. Difference between successor and kth smallest?
3. How to recover BST in O(1) space? (Morris inorder — advanced)

### Bonus — Range Sum (938)

```js
function rangeSumBST(root, low, high) {
  if (!root) return 0;
  let sum = root.val >= low && root.val <= high ? root.val : 0;
  if (root.val > low) sum += rangeSumBST(root.left, low, high);
  if (root.val < high) sum += rangeSumBST(root.right, low, high);
  return sum;
}
```

---

*End of Day 42 LeetCode — Week 6 complete*
