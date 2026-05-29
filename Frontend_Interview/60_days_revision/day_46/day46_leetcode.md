# Day 46 — LeetCode (Radix & Lex Order)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 1804 | [Implement Trie II (Prefix Tree)](#1-implement-trie-ii-prefix-tree-1804) | Medium | Count words/prefixes |
| 440 | [K-th Smallest in Lex Order](#2-k-th-smallest-in-lex-order-440) | Hard | Digit trie count |
| 386 | [Lexicographical Numbers](#3-lexicographical-numbers-386) | Medium | DFS digit trie |

---

## Table of Contents

1. [Implement Trie II (1804)](#1-implement-trie-ii-prefix-tree-1804)
2. [K-th Smallest in Lex Order (440)](#2-k-th-smallest-in-lex-order-440)
3. [Lexicographical Numbers (386)](#3-lexicographical-numbers-386)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Implement Trie II (Prefix Tree) (1804)

### Problem (short)

Implement trie with `insert`, `erase`, `countWordsEqualTo`, `countWordsStartingWith`.

### Hint 1 — Dual counters

`wordCount` at terminal; `prefixCount` on every node along path.

### Hint 2 — erase

Mirror insert — decrement counts; don't delete nodes (lazy).

### Solution — JavaScript

```js
class TrieNodeII {
  constructor() {
    this.children = new Map();
    this.wordCount = 0;
    this.prefixCount = 0;
  }
}

class TrieII {
  constructor() {
    this.root = new TrieNodeII();
  }

  insert(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNodeII());
      node = node.children.get(ch);
      node.prefixCount++;
    }
    node.wordCount++;
  }

  _walk(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) return null;
      node = node.children.get(ch);
    }
    return node;
  }

  countWordsEqualTo(word) {
    const node = this._walk(word);
    return node ? node.wordCount : 0;
  }

  countWordsStartingWith(prefix) {
    const node = this._walk(prefix);
    return node ? node.prefixCount : 0;
  }

  erase(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) return;
      node = node.children.get(ch);
      node.prefixCount--;
    }
    node.wordCount--;
  }
}
```

| Operation | Time | Space |
|-----------|------|-------|
| All ops | O(m) | O(total chars) |

---

## 2. K-th Smallest in Lex Order (440)

### Problem (short)

Integers 1..n in lex order — find k-th without building full list.

```
n = 13, k = 2 → 10  (order: 1,10,11,12,13,2,3,...)
```

### Hint 1 — Digit trie mental model

Lex order = preorder DFS on 10-ary tree of digits.

### Hint 2 — countSteps

Count numbers in subtree `[cur, cur+1)` range within `[1, n]`.

### Solution — JavaScript

```js
/**
 * @param {number} n
 * @param {number} k
 * @return {number}
 */
function findKthNumber(n, k) {
  let cur = 1;
  k--;

  while (k > 0) {
    const steps = countBetween(n, cur, cur + 1);
    if (steps <= k) {
      cur++;
      k -= steps;
    } else {
      cur *= 10;
      k--;
    }
  }
  return cur;
}

function countBetween(n, first, last) {
  let steps = 0;
  while (first <= n) {
    steps += Math.min(n + 1, last) - first;
    first *= 10;
    last *= 10;
  }
  return steps;
}
```

| Time | Space |
|------|-------|
| O(log² n) | O(1) |

---

## 3. Lexicographical Numbers (386)

### Problem (short)

Return integers 1..n in lexicographical order.

```
n = 13
Output: [1,10,11,12,13,2,3,4,5,6,7,8,9]
```

### Hint 1 — Same as 440 DFS

Preorder on digit trie from 1.

### Hint 2 — Avoid cur=0

Start DFS at 1; append digits 0-9.

### Solution — JavaScript

```js
/**
 * @param {number} n
 * @return {number[]}
 */
function lexicalOrder(n) {
  const result = [];

  const dfs = (num) => {
    if (num > n) return;
    result.push(num);
    for (let d = 0; d <= 9; d++) {
      const next = num * 10 + d;
      if (num === 0 && d === 0) continue;
      dfs(next);
    }
  };

  for (let i = 1; i <= 9; i++) dfs(i);
  return result;
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) excl. output |

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **1804** | wordCount + prefixCount | Lazy erase decrement |
| **440** | Digit trie + countSteps | Skip/advance k |
| **386** | Preorder DFS digits | 1..9 then append 0-9 |

### Follow-ups

| Question | Answer |
|----------|--------|
| 1804 hard delete? | Post-order prune when counts hit 0 |
| 440 generate all? | DFS O(n) — 440 avoids full list |
| 386 iterative? | Stack simulating DFS |

### Test cases

```js
const t2 = new TrieII();
t2.insert("apple");
t2.insert("apple");
console.assert(t2.countWordsEqualTo("apple") === 2);
console.assert(t2.countWordsStartingWith("app") === 2);
t2.erase("apple");
console.assert(t2.countWordsEqualTo("apple") === 1);

console.assert(findKthNumber(13, 2) === 10);
console.assert(lexicalOrder(13).join(",") === "1,10,11,12,13,2,3,4,5,6,7,8,9");
```

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 1804 | O(m) | O(N) |
| 440 | O(log² n) | O(1) |
| 386 | O(n) | O(1) |

---

*End of Day 46 LeetCode*
