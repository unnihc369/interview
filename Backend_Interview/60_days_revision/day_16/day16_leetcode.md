# Day 16 — BST Patterns

**Topics:** Validate BST · Kth Smallest in BST · Serialize/Deserialize Binary Tree

---

# 1. Validate Binary Search Tree

## Problem Statement

Given the root of a binary tree, determine if it is a valid BST.

BST property: for every node, all values in left subtree < node.val < all values in right subtree.

---

# Example

```text
Input:
    2
   / \
  1   3
Output: true

Input:
    5
   / \
  1   4
     / \
    3   6
Output: false (4 is in right of 5 but 4 < 5)
```

---

# Brute Force

For each node, check entire left subtree < node and entire right subtree > node.

```text
O(n^2)
```

---

# Optimized Approach (In-order / Range Check)

## Core Idea — Range Validation

Pass valid range `(min, max)` down the tree:

- Left child: `(min, node.val)`
- Right child: `(node.val, max)`

---

# Java Solution

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return validate(root, null, null);
    }

    private boolean validate(TreeNode node, Integer min, Integer max) {
        if (node == null) return true;
        if ((min != null && node.val <= min) || (max != null && node.val >= max)) {
            return false;
        }
        return validate(node.left, min, node.val)
            && validate(node.right, node.val, max);
    }
}
```

---

# JavaScript Solution

```javascript
function isValidBST(root) {
  function validate(node, min, max) {
    if (!node) return true;
    if ((min !== null && node.val <= min) || (max !== null && node.val >= max)) {
      return false;
    }
    return validate(node.left, min, node.val) &&
           validate(node.right, node.val, max);
  }
  return validate(root, null, null);
}
```

---

# Time Complexity

```text
O(n)
```

---

# 2. Kth Smallest Element in a BST

## Problem Statement

Given root of BST and integer `k`, return the k-th smallest value (1-indexed).

---

# Example

```text
Input:
    3
   / \
  1   4
   \
    2
k = 1 → Output: 1
```

---

# Brute Force

In-order traversal, store all values, return k-th.

```text
O(n) time, O(n) space
```

---

# Optimized Approach (In-order Iterative)

## Core Idea

In-order traversal of BST gives sorted order. Stop at k-th element.

---

# Java Solution

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode curr = root;

        while (curr != null || !stack.isEmpty()) {
            while (curr != null) {
                stack.push(curr);
                curr = curr.left;
            }
            curr = stack.pop();
            k--;
            if (k == 0) return curr.val;
            curr = curr.right;
        }
        return -1;
    }
}
```

---

# Follow-up: What if tree is modified frequently?

Augment nodes with subtree size. Navigate: if left.size >= k → go left; else if left.size + 1 == k → current; else go right with k -= left.size + 1. `O(h)` per query.

---

# JavaScript Solution

```javascript
function kthSmallest(root, k) {
  const stack = [];
  let curr = root;
  while (curr || stack.length) {
    while (curr) { stack.push(curr); curr = curr.left; }
    curr = stack.pop();
    k--;
    if (k === 0) return curr.val;
    curr = curr.right;
  }
}
```

---

# Time Complexity

```text
O(h + k) average, O(n) worst
```

---

# 3. Serialize and Deserialize Binary Tree

## Problem Statement

Design an algorithm to serialize a binary tree to a string and deserialize the string back to the original tree structure.

---

# Example

```text
Input:
    1
   / \
  2   3
     / \
    4   5

Serialized: "1,2,#,#,3,4,#,#,5,#,#"
Output: same tree
```

---

# Brute Force

Store in-order + pre-order separately. Reconstruct — fragile if values duplicate.

---

# Optimized Approach (BFS / Pre-order with Null Markers)

## Core Idea — Pre-order DFS

Serialize: `nodeVal,left,right` with `#` for null.

Deserialize: read token, if `#` return null, else create node and recurse.

---

# Java Solution

```java
public class Codec {
    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        serializeDFS(root, sb);
        return sb.toString();
    }

    private void serializeDFS(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.append("#,");
            return;
        }
        sb.append(node.val).append(",");
        serializeDFS(node.left, sb);
        serializeDFS(node.right, sb);
    }

    public TreeNode deserialize(String data) {
        Queue<String> queue = new LinkedList<>(Arrays.asList(data.split(",")));
        return deserializeDFS(queue);
    }

    private TreeNode deserializeDFS(Queue<String> queue) {
        String val = queue.poll();
        if ("#".equals(val)) return null;
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left = deserializeDFS(queue);
        node.right = deserializeDFS(queue);
        return node;
    }
}
```

---

# JavaScript Solution

```javascript
function serialize(root) {
  const result = [];
  function dfs(node) {
    if (!node) { result.push('#'); return; }
    result.push(String(node.val));
    dfs(node.left);
    dfs(node.right);
  }
  dfs(root);
  return result.join(',');
}

function deserialize(data) {
  const tokens = data.split(',');
  let i = 0;
  function dfs() {
    if (tokens[i] === '#') { i++; return null; }
    const node = new TreeNode(Number(tokens[i++]));
    node.left = dfs();
    node.right = dfs();
    return node;
  }
  return dfs();
}
```

---

# Time Complexity

```text
O(n) serialize and deserialize
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Validate BST | Range (min, max) propagation | O(n) |
| Kth Smallest BST | In-order traversal (iterative) | O(h+k) |
| Serialize BT | Pre-order DFS with null markers | O(n) |

---

# Interview Questions

## Why range check beats comparing only parent-child?

Parent-child check fails when a deep node violates BST property but immediate children look valid.

## Can you use Morris traversal for O(1) space kth smallest?
Yes — advanced follow-up. Thread tree during in-order, no stack.

## BFS vs pre-order for serialization?
Both work. Pre-order with nulls is simpler to deserialize recursively. BFS uses level-order with null placeholders (LeetCode style).

---

*End of Day 16 LeetCode*
