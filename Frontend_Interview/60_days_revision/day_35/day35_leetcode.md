# Day 35 — LeetCode (Trees Mock Revision)

**Timed mock (45 min):** Solve without looking at hints first, then verify below.

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 102 | [Binary Tree Level Order Traversal](#1-mock-102-level-order) | Medium | BFS |
| 236 | [Lowest Common Ancestor](#2-mock-236-lca) | Medium | Post-order |
| 124 | [Binary Tree Maximum Path Sum](#3-mock-124-max-path-sum) | Hard | DFS global |

---

## 1. Mock — 102 Level Order

### Self-check (5 min after coding)

- [ ] Empty tree → `[]`
- [ ] Single node → `[[val]]`
- [ ] Used `size = queue.length` before inner loop

### Solution

```js
function levelOrder(root) {
  if (!root) return [];
  const res = [], q = [root];
  while (q.length) {
    const size = q.length, level = [];
    for (let i = 0; i < size; i++) {
      const n = q.shift();
      level.push(n.val);
      if (n.left) q.push(n.left);
      if (n.right) q.push(n.right);
    }
    res.push(level);
  }
  return res;
}
```

**Target time:** 10 min

---

## 2. Mock — 236 LCA

### Self-check

- [ ] Handles p or q being ancestor of the other
- [ ] Returns node when found in subtree

### Solution

```js
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;
  const L = lowestCommonAncestor(root.left, p, q);
  const R = lowestCommonAncestor(root.right, p, q);
  if (L && R) return root;
  return L || R;
}
```

**Target time:** 12 min

---

## 3. Mock — 124 Max Path Sum

### Self-check

- [ ] Negative values handled
- [ ] `Math.max(childGain, 0)` for arms
- [ ] Global max updated with L + R + node.val
- [ ] Return value is single arm only

### Solution

```js
function maxPathSum(root) {
  let max = -Infinity;
  function gain(node) {
    if (!node) return 0;
    const L = Math.max(gain(node.left), 0);
    const R = Math.max(gain(node.right), 0);
    max = Math.max(max, node.val + L + R);
    return node.val + Math.max(L, R);
  }
  gain(root);
  return max;
}
```

**Target time:** 20 min

---

## Mock Scoring Rubric

| Score | Criteria |
|-------|----------|
| **Pass** | All 3 correct + explain complexity |
| **Partial** | 2/3 with clean code |
| **Review** | Re-study Days 29–34 patterns |

### Verbal follow-ups (practice aloud)

1. Why BFS space is O(w) not O(n)?
2. Difference between 236 and 235?
3. Can 124 be solved iteratively?

---

*End of Day 35 LeetCode — Week 5 complete*
