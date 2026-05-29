# Day 43 — LeetCode (Trie Basics)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 208 | [Implement Trie (Prefix Tree)](#1-implement-trie-prefix-tree-208) | Medium | Core trie |
| 211 | [Design Add and Search Words Data Structure](#2-design-add-and-search-words-211) | Medium | Trie + DFS wildcard |
| 212 | [Word Search II](#3-word-search-ii-212) | Hard | Board DFS + trie pruning |

---

## Table of Contents

1. [Implement Trie (Prefix Tree) (208)](#1-implement-trie-prefix-tree-208)
2. [Design Add and Search Words (211)](#2-design-add-and-search-words-211)
3. [Word Search II (212)](#3-word-search-ii-212)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Implement Trie (Prefix Tree) (208)

### Problem (short)

Implement `Trie` with `insert`, `search`, and `startsWith`.

```
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // true
trie.search("app");     // false
trie.startsWith("app"); // true
```

### Hint 1 — Node shape

Map children + `isEnd` boolean. Walk character by character.

### Hint 2 — search vs startsWith

`search` requires terminal node (`isEnd`). `startsWith` only needs path exists.

### Solution — JavaScript (Map)

```js
class TrieNode {
  constructor() {
    this.children = new Map();
    this.isEnd = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }

  insert(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNode());
      node = node.children.get(ch);
    }
    node.isEnd = true;
  }

  _walk(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) return null;
      node = node.children.get(ch);
    }
    return node;
  }

  search(word) {
    const node = this._walk(word);
    return node !== null && node.isEnd;
  }

  startsWith(prefix) {
    return this._walk(prefix) !== null;
  }
}
```

### Solution — Array variant (a–z)

```js
class TrieNodeArr {
  constructor() {
    this.children = new Array(26).fill(null);
    this.isEnd = false;
  }
  idx(c) { return c.charCodeAt(0) - 97; }
}
```

| Operation | Time | Space |
|-----------|------|-------|
| insert / search / startsWith | O(m) | O(total chars) |

---

## 2. Design Add and Search Words (211)

### Problem (short)

Support `addWord(word)` and `search(word)` where `.` matches any single letter.

```
addWord("bad"), addWord("dad"), addWord("mad")
search("pad") → false
search(".ad") → true
search("b..") → true
```

### Hint 1 — Wildcard = branch

On `.`, try all children recursively (DFS).

### Hint 2 — Pruning

Return true on first successful branch; false if all fail.

### Solution — JavaScript

```js
class WordDictionary {
  constructor() {
    this.root = new TrieNode();
  }

  addWord(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNode());
      node = node.children.get(ch);
    }
    node.isEnd = true;
  }

  search(word) {
    return this._dfs(this.root, word, 0);
  }

  _dfs(node, word, i) {
    if (i === word.length) return node.isEnd;
    const ch = word[i];
    if (ch === ".") {
      for (const child of node.children.values()) {
        if (this._dfs(child, word, i + 1)) return true;
      }
      return false;
    }
    if (!node.children.has(ch)) return false;
    return this._dfs(node.children.get(ch), word, i + 1);
  }
}
```

| addWord | search | Space |
|---------|--------|-------|
| O(m) | O(26^m) worst with dots | O(total chars) |

**Interview tip:** Mention trie beats scanning all words O(n×m) for each query.

---

## 3. Word Search II (212)

### Problem (short)

Given `board` (2D chars) and `words[]`, return all words found by adjacent cell paths (no reuse per path).

```
board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]]
words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]
```

### Hint 1 — Trie all words

Build trie from `words`. Store full word at terminal node.

### Hint 2 — Prune trie branches

After finding a word, remove that branch or mark visited to avoid duplicates.

### Hint 3 — Backtrack

Standard board DFS with `visited` set; undo on return.

### Solution — JavaScript

```js
/**
 * @param {character[][]} board
 * @param {string[]} words
 * @return {string[]}
 */
function findWords(board, words) {
  const root = new TrieNode();
  for (const w of words) {
    let node = root;
    for (const ch of w) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNode());
      node = node.children.get(ch);
    }
    node.isEnd = true;
    node.word = w;
  }

  const rows = board.length, cols = board[0].length;
  const result = new Set();

  const dfs = (r, c, node) => {
    if (node.isEnd) {
      result.add(node.word);
      node.isEnd = false; // avoid duplicate paths
    }
    if (r < 0 || c < 0 || r >= rows || c >= cols) return;
    const ch = board[r][c];
    if (!node.children.has(ch)) return;

    board[r][c] = "#";
    const next = node.children.get(ch);
    dfs(r + 1, c, next);
    dfs(r - 1, c, next);
    dfs(r, c + 1, next);
    dfs(r, c - 1, next);
    board[r][c] = ch;

    // prune empty branch
    if (next.children.size === 0 && !next.isEnd) {
      node.children.delete(ch);
    }
  };

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      dfs(r, c, root);
    }
  }
  return [...result];
}
```

| Time | Space |
|------|-------|
| O(m × n × 4^L) with trie pruning | O(total chars in words) |

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **208** | Map trie | insert / walk / isEnd |
| **211** | Trie + DFS | `.` → try all children |
| **212** | Board DFS + trie | Prune dead branches |

### Follow-ups

| Question | Answer |
|----------|--------|
| 208 array vs map? | Array for a–z; Map for general |
| 211 optimize many dots? | Still branch; trie helps vs word list |
| 212 why store word at node? | Avoid rebuilding string on match |
| 212 duplicate words? | Use Set or clear isEnd after add |

### Test cases

```js
const t = new Trie();
t.insert("apple");
console.assert(t.search("apple") && t.startsWith("app") && !t.search("app"));

const wd = new WordDictionary();
wd.addWord("bad");
console.assert(wd.search(".ad") && wd.search("b.."));

findWords(
  [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]],
  ["oath","pea","eat","rain"]
); // ["eat","oath"] (order may vary)
```

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 208 | O(m) per op | O(N) |
| 211 | O(m) add; O(26^k) search | O(N) |
| 212 | O(R×C×4^L) pruned | O(N) |

---

*End of Day 43 LeetCode*
