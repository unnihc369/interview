# Day 49 — LeetCode (Timed Mock: Tries)

**Format:** Timed practice — 3 problems · **45 min total** · No peeking at solutions until attempt complete

| # | Problem | Difficulty | Target time |
|---|---------|------------|-------------|
| 208 | [Implement Trie (Prefix Tree)](#mock-1--implement-trie-208) | Medium | 12 min |
| 211 | [Design Add and Search Words](#mock-2--add-search-word-211) | Medium | 15 min |
| 212 | [Word Search II](#mock-3--word-search-ii-212) | Hard | 18 min |

---

## Table of Contents

1. [Mock Rules & Scoring](#1-mock-rules--scoring)
2. [Mock 1 — Implement Trie (208)](#mock-1--implement-trie-208)
3. [Mock 2 — Add Search Word (211)](#mock-2--add-search-word-211)
4. [Mock 3 — Word Search II (212)](#mock-3--word-search-ii-212)
5. [Post-Mock Review Checklist](#5-post-mock-review-checklist)
6. [Solutions (Reveal After Attempt)](#6-solutions-reveal-after-attempt)
7. [Week 7 Pattern Cheat Sheet](#7-week-7-pattern-cheat-sheet)

---

## 1. Mock Rules & Scoring

### Before you start

1. Timer: **45 minutes** total.
2. Implement `Trie` / `WordDictionary` as **classes**.
3. For 212: build trie first, then board DFS — say prune strategy aloud.

### Scoring rubric

| Criteria | Points |
|----------|--------|
| Correct trie node structure | 3 |
| Working code | 3 |
| Complexity stated | 2 |
| Edge cases | 2 |

| Total | Grade |
|-------|-------|
| 25–30 | Trie patterns mastered — Week 7 done |
| 18–24 | Review Days 43–45 |
| < 18 | Re-read trie cheat sheets |

---

## Mock 1 — Implement Trie (208)

### Problem (short)

Implement `Trie` with `insert(word)`, `search(word)`, `startsWith(prefix)`.

```
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // true
trie.search("app");     // false
trie.startsWith("app"); // true
```

### Hints (if stuck 5+ min)

<details>
<summary>Hint 1</summary>
Map children + isEnd boolean.
</details>

<details>
<summary>Hint 2</summary>
search checks isEnd; startsWith only checks path exists.
</details>

### Your workspace

```js
class TrieNode {
  constructor() {
    // YOUR CODE
  }
}

class Trie {
  constructor() {
    // YOUR CODE
  }
  insert(word) {
    // YOUR CODE
  }
  search(word) {
    // YOUR CODE
  }
  startsWith(prefix) {
    // YOUR CODE
  }
}
```

### Tests

```js
const t = new Trie();
t.insert("apple");
console.assert(t.search("apple") === true);
console.assert(t.search("app") === false);
console.assert(t.startsWith("app") === true);
t.insert("app");
console.assert(t.search("app") === true);
```

---

## Mock 2 — Add Search Word (211)

### Problem (short)

`addWord(word)` and `search(word)` where `.` matches any letter.

```
addWord("bad"), addWord("dad"), addWord("mad")
search("pad") → false
search(".ad") → true
search("b..") → true
```

### Hints

<details>
<summary>Hint 1</summary>
Same trie as 208.
</details>

<details>
<summary>Hint 2</summary>
On `.`, DFS try ALL children. On letter, single branch.
</details>

### Your workspace

```js
class WordDictionary {
  constructor() {
    // YOUR CODE
  }
  addWord(word) {
    // YOUR CODE
  }
  search(word) {
    // YOUR CODE
  }
}
```

### Tests

```js
const wd = new WordDictionary();
wd.addWord("bad");
wd.addWord("dad");
wd.addWord("mad");
console.assert(wd.search("pad") === false);
console.assert(wd.search(".ad") === true);
console.assert(wd.search("b..") === true);
console.assert(wd.search("...") === true);
```

---

## Mock 3 — Word Search II (212)

### Problem (short)

2D board + word list. Return all words formable by adjacent paths (no cell reuse per path).

```
board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]]
words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]
```

### Hints

<details>
<summary>Hint 1</summary>
Build trie from all words. Store full word at terminal node.
</details>

<details>
<summary>Hint 2</summary>
DFS from each cell. Mark visited with `#`. Backtrack restore.
</details>

<details>
<summary>Hint 3</summary>
Prune: delete trie branch when child has no children and not isEnd.
</details>

### Your workspace

```js
/**
 * @param {character[][]} board
 * @param {string[]} words
 * @return {string[]}
 */
function findWords(board, words) {
  // YOUR CODE
}
```

### Tests

```js
const r = findWords(
  [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]],
  ["oath","pea","eat","rain"]
);
console.assert(r.length === 2 && r.includes("eat") && r.includes("oath"));

const r2 = findWords([["a"]], ["a"]);
console.assert(r2[0] === "a");
```

---

## 5. Post-Mock Review Checklist

- [ ] Used Map for children (or array with index helper)?
- [ ] 208: distinguished search vs startsWith (isEnd)?
- [ ] 211: DFS on `.` tries all children?
- [ ] 212: stored word at terminal node?
- [ ] 212: pruned dead trie branches?
- [ ] Stated O(m) insert and O(m) search complexity?

---

## 6. Solutions (Reveal After Attempt)

### 208 — Implement Trie

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

  _walk(s) {
    let node = this.root;
    for (const ch of s) {
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

| Operation | Time | Space |
|-----------|------|-------|
| All | O(m) | O(total chars) |

### 211 — Add Search Word

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

| addWord | search |
|---------|--------|
| O(m) | O(26^k × m) worst |

### 212 — Word Search II

```js
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
      node.isEnd = false;
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

    if (!next.isEnd && next.children.size === 0) {
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
| O(R×C×4^L) pruned | O(N) |

---

## 7. Week 7 Pattern Cheat Sheet

| Day | Pattern | Key LC |
|-----|---------|--------|
| 43 | Core trie | 208, 211, 212 |
| 44 | Dict prefix | 648, 677, 720 |
| 45 | Delete/stream | 1268, 1032, 676 |
| 46 | Radix/lex | 1804, 440, 386 |
| 47 | Suffix/dup | 745, 1062, 1044 |
| 48 | Binary XOR | 421, 1707, 1938 |
| 49 | **Mock** | 208, 211, 212 |

### One-liners

| Problem | Pattern |
|---------|---------|
| **208** | Map + isEnd + walk |
| **211** | `.` → DFS all children |
| **212** | Trie on board + prune branches |

---

## Week 7 Complete

Congratulations — 7 days of Tries covered. Next week continues the 60-day plan.

**Self-grade today:**

| Score | Action |
|-------|--------|
| 25–30 | Move to Week 8 |
| 18–24 | Redo Day 43 + 45 leetcode untimed |
| < 18 | Re-read all 7 concept cheat sheets |

---

*End of Day 49 LeetCode mock — Week 7 Tries complete*
