# Day 29 — LeetCode (Trees — Recursion Basics)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 104 | [Maximum Depth of Binary Tree](#1-maximum-depth-of-binary-tree-104) | Easy | Recursion / BFS |
| 226 | [Invert Binary Tree](#2-invert-binary-tree-226) | Easy | Recursion / swap |
| 124 | [Binary Tree Maximum Path Sum](#3-binary-tree-maximum-path-sum-124) | Hard | DFS + global max |

---

## Table of Contents

1. [Maximum Depth of Binary Tree (104)](#1-maximum-depth-of-binary-tree-104)
2. [Invert Binary Tree (226)](#2-invert-binary-tree-226)
3. [Binary Tree Maximum Path Sum (124)](#3-binary-tree-maximum-path-sum-124)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Maximum Depth of Binary Tree (104)

### Problem

Return the **maximum depth** (number of nodes along longest root-to-leaf path).

```
Input:  [3,9,20,null,null,15,7]
Output: 3
```

### Hint 1 — Recursive definition

Depth of tree = `1 + max(depth(left), depth(right))`.  
Empty tree → depth `0`.

### Hint 2 — BFS alternative

Count levels while processing queue level-by-level.

### JavaScript — Recursive

```js
/**
 * @param {TreeNode|null} root
 * @return {number}
 */
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

### JavaScript — Iterative BFS

```js
function maxDepthBFS(root) {
  if (!root) return 0;
  const queue = [root];
  let depth = 0;

  while (queue.length) {
    const size = queue.length;
    depth++;
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  return depth;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) recursive / O(w) BFS |

---

## 2. Invert Binary Tree (226)

### Problem

Swap left and right of **every** node. Return inverted root.

```
Input:     4              Output:    4
         /   \                      /   \
        2     7                    7     2
       / \   / \                  / \   / \
      1   3 6   9                9   6 3   1
```

### Hint — Post-order swap

At each node: invert left subtree, invert right subtree, then swap pointers.

### JavaScript

```js
function invertTree(root) {
  if (!root) return null;

  const left = invertTree(root.left);
  const right = invertTree(root.right);

  root.left = right;
  root.right = left;
  return root;
}
```

### Iterative (stack)

```js
function invertTreeIterative(root) {
  if (!root) return null;
  const stack = [root];
  while (stack.length) {
    const node = stack.pop();
    [node.left, node.right] = [node.right, node.left];
    if (node.left) stack.push(node.left);
    if (node.right) stack.push(node.right);
  }
  return root;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 3. Binary Tree Maximum Path Sum (124)

### Problem

**Path** = any sequence of nodes where each pair is connected by an edge (no need to pass through root).  
Return **maximum path sum** (node values can be negative).

```
Input:  [-10,9,20,null,null,15,7]
Output: 42  (path 15 → 20 → 7)
```

### Hint 1 — What can a node contribute?

At node `n`, a path through `n` can use:
- `n.val + leftGain` (if positive)
- `n.val + rightGain` (if positive)
- `n.val + leftGain + rightGain` → **candidate for global max** (arch through n)

### Hint 2 — Return value vs global

DFS returns **max single-arm gain** upward (one branch only).  
Track `globalMax` separately for arch paths.

### Walkthrough

```
At node 20: leftGain=15, rightGain=7
Arch sum = 15+20+7 = 42 → update global
Return to parent = 20 + max(15,7) = 35
```

### JavaScript

```js
function maxPathSum(root) {
  let globalMax = -Infinity;

  function gain(node) {
    if (!node) return 0;

    const left = Math.max(gain(node.left), 0);
    const right = Math.max(gain(node.right), 0);

    globalMax = Math.max(globalMax, node.val + left + right);
    return node.val + Math.max(left, right);
  }

  gain(root);
  return globalMax;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

### Common mistakes

- Forgetting `Math.max(child, 0)` — negative arms should be ignored
- Returning arch sum to parent (must return single arm only)

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | Key idea |
|---------|---------|----------|
| **104** | Recursion / BFS | `1 + max(L,R)` or level count |
| **226** | Post-order swap | Swap children after recursing |
| **124** | DFS + global | Return one arm; track arch max |

### Solve order

1. **104** — Warm-up recursion
2. **226** — Mutate tree / swap
3. **124** — Global variable + return semantics

---

*End of Day 29 LeetCode*
