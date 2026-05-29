# Day 15 — Binary Tree Patterns

**Topics:** Level Order Traversal · Lowest Common Ancestor · Max Path Sum

---

# 1. Binary Tree Level Order Traversal

## Problem Statement

Given the root of a binary tree, return the level-order traversal as a list of lists — each inner list contains node values at that depth, left to right.

---

# Example

```text
Input:
    3
   / \
  9  20
    /  \
   15   7

Output:
[[3], [9, 20], [15, 7]]
```

---

# Brute Force Approach

DFS with depth tracking, then group by depth.

```text
O(n) time, O(n) space
```

Works but BFS is cleaner for level-by-level.

---

# Optimized Approach (BFS)

## Core Idea

1. Use a queue; enqueue root
2. Process all nodes at current level (track `size`)
3. Enqueue children for next level
4. Repeat until queue empty

---

# Java Solution

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            result.add(level);
        }
        return result;
    }
}
```

---

# JavaScript Solution

```javascript
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];

  while (queue.length) {
    const size = queue.length;
    const level = [];
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}
```

---

# Time Complexity

```text
O(n) — visit every node once
```

---

# 2. Lowest Common Ancestor of a Binary Tree

## Problem Statement

Given a binary tree and two nodes `p` and `q`, return their lowest common ancestor (LCA) — the deepest node that has both `p` and `q` as descendants (a node can be its own descendant).

---

# Example

```text
Input:
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

p = 5, q = 1  →  Output: 3
p = 5, q = 4  →  Output: 5
```

---

# Brute Force

Store path from root to `p` and root to `q`. Find last common node.

```text
O(n) time, O(n) space
```

---

# Optimized Approach (Recursive DFS)

## Core Idea

Post-order recursion:

- If node is null, or equals `p` or `q`, return node
- Recurse left and right
- If both sides return non-null → current node is LCA
- If only one side non-null → propagate that side up

---

# Java Solution

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;

        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if (left != null && right != null) return root;
        return left != null ? left : right;
    }
}
```

---

# JavaScript Solution

```javascript
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) return root;
  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);
  if (left && right) return root;
  return left || right;
}
```

---

# Time Complexity

```text
O(n)
```

---

# Interview Follow-up

**LCA in BST?** Use BST property: if both values smaller → go left; both larger → go right; else current node is LCA. `O(h)` time.

---

# 3. Binary Tree Maximum Path Sum

## Problem Statement

A path is any sequence of nodes where each pair of adjacent nodes has an edge. Path sum = sum of node values. Return the maximum path sum (path need not pass through root).

---

# Example

```text
Input:
      -10
      /  \
     9   20
        /  \
       15   7

Output: 42  (path 15 → 20 → 7)
```

---

# Brute Force

Try every node as path endpoint. Exponential — too slow.

---

# Optimized Approach (Post-order DFS)

## Core Idea

At each node, compute:

1. **Max gain through this node** (one branch up to parent): `node.val + max(0, leftGain, rightGain)`
2. **Max path through this node** (left + node + right, no parent): `node.val + max(0, leftGain) + max(0, rightGain)`

Track global max across all nodes.

---

# Java Solution

```java
class Solution {
    private int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        maxGain(root);
        return maxSum;
    }

    private int maxGain(TreeNode node) {
        if (node == null) return 0;

        int left = Math.max(maxGain(node.left), 0);
        int right = Math.max(maxGain(node.right), 0);

        maxSum = Math.max(maxSum, node.val + left + right);
        return node.val + Math.max(left, right);
    }
}
```

# JavaScript Solution

```javascript
function maxPathSum(root) {
  let maxSum = -Infinity;
  function maxGain(node) {
    if (!node) return 0;
    const left = Math.max(maxGain(node.left), 0);
    const right = Math.max(maxGain(node.right), 0);
    maxSum = Math.max(maxSum, node.val + left + right);
    return node.val + Math.max(left, right);
  }
  maxGain(root);
  return maxSum;
}
```

---

# Time Complexity

```text
O(n) — single DFS pass
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Level Order | BFS with level-size loop | O(n) |
| LCA BT | Post-order recursive split | O(n) |
| Max Path Sum | Post-order + global max | O(n) |

---

# Interview Questions

## Why BFS over DFS for level order?
Interviewer wants to see queue discipline. DFS works but requires sorting by depth; BFS is natural.

## Can LCA work with parent pointers?
Yes — store parent map during tree construction. Useful when interviewer gives follow-up: "What if nodes have parent pointers?"

## Max path sum — why not reuse diameter of tree?
Diameter only handles positive weights. Max path sum allows negative nodes — must use gain clipping (ignore negative branches).

---

*End of Day 15*
