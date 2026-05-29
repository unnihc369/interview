# Day 45 — Trie Deletion & Advanced Search

**Week 7 — Tries** · **Topics:** Lazy deletion · Hard delete · Word games · Magic dictionary · Stream matching

---

## Table of Contents

1. [Trie Deletion Strategies](#1-trie-deletion-strategies)
2. [Lazy vs Hard Delete](#2-lazy-vs-hard-delete)
3. [Word Game Prefix/Suffix](#3-word-game-prefixsuffix)
4. [Magic Dictionary (LC 676)](#4-magic-dictionary-lc-676)
5. [Stream of Characters (LC 1032)](#5-stream-of-characters-lc-1032)
6. [Search Suggestions (LC 1268)](#6-search-suggestions-lc-1268)
7. [Predictive Text Architecture](#7-predictive-text-architecture)
8. [Interview Q&A](#8-interview-qa)

---

## 1. Trie Deletion Strategies

Removing `"apple"` from a trie shared with `"app"` and `"application"`:

```
Before:  a → p → p → l → e* (and branch l → … → n*)
After:   a → p → p*  (keep "app", prune "le" if unused)
```

Two approaches: **lazy** (mark inactive) vs **hard** (remove nodes).

---

## 2. Lazy vs Hard Delete

### Lazy delete (production-friendly)

```js
  deleteLazy(word) {
    const node = this._walk(word);
    if (!node || !node.isEnd) return false;
    node.isEnd = false;
    // Optionally decrement prefix counts along path
    let n = this.root;
    for (const ch of word) {
      n = n.children.get(ch);
      n.count = (n.count || 1) - 1;
    }
    return true;
  }

  search(word) {
    const node = this._walk(word);
    return node !== null && node.isEnd; // deleted words return false
  }
```

| Pros | Cons |
|------|------|
| O(m) simple | Orphan nodes remain — memory leak over time |
| Safe with shared prefixes | Periodic compaction needed |

### Hard delete (interview classic)

```js
  deleteHard(word) {
    this._remove(this.root, word, 0);
  }

  _remove(node, word, i) {
    if (i === word.length) {
      if (!node.isEnd) return false;
      node.isEnd = false;
      return node.children.size === 0;
    }
    const ch = word[i];
    if (!node.children.has(ch)) return false;
    const child = node.children.get(ch);
    const shouldDeleteChild = this._remove(child, word, i + 1);
    if (shouldDeleteChild) node.children.delete(ch);
    return !node.isEnd && node.children.size === 0;
  }
```

**Return value:** `true` if current node can be removed (no children, not end).

---

## 3. Word Game Prefix/Suffix

### Prefix game (Wordle-style filtering)

```js
function filterByClues(words, green, yellow, gray) {
  // green: [{ pos, char }]
  // yellow: [{ char, notPos }]
  // gray: Set of chars
  return words.filter((w) => {
    for (const { pos, char } of green) if (w[pos] !== char) return false;
    for (const { char, notPos } of yellow) {
      if (!w.includes(char) || w[notPos] === char) return false;
    }
    for (const ch of gray) {
      if (w.includes(ch) && !green.some(g => g.char === ch) && !yellow.some(y => y.char === ch))
        return false;
    }
    return true;
  });
}
```

**Trie optimization:** Bucket words by first letter / length in trie subtrees.

### Suffix queries

Standard trie on **reversed words** → prefix on reversed = suffix on original.

```js
function buildSuffixTrie(words) {
  const trie = new Trie();
  for (const w of words) trie.insert(w.split("").reverse().join(""));
  return trie;
}

function hasSuffix(word, suffix) {
  return trie.startsWith(suffix.split("").reverse().join(""));
}
```

---

## 4. Magic Dictionary (LC 676)

Build dictionary; `search(word)` returns true if **exactly one** character can be changed to match a dict word.

```js
class MagicDictionary {
  constructor() {
    this.words = [];
  }
  buildDict(dict) { this.words = dict; }

  search(word) {
    const dfs = (i, diff) => {
      if (i === word.length) return diff === 1;
      for (const w of this.words) {
        if (w.length !== word.length) continue;
        // optimize: trie + one-mismatch walk
      }
    };
    // Trie approach:
    return this._searchTrie(this.root, word, 0, false);
  }

  _searchTrie(node, word, i, used) {
    if (i === word.length) return used && node.isEnd;
    const ch = word[i];
    if (node.children.has(ch)) {
      if (this._searchTrie(node.children.get(ch), word, i + 1, used)) return true;
    }
    if (used) return false;
    for (const [c, child] of node.children) {
      if (c !== ch && this._searchTrie(child, word, i + 1, true)) return true;
    }
    return false;
  }
}
```

---

## 5. Stream of Characters (LC 1032)

Queries arrive as stream; return true if **any suffix** of stream-so-far matches a dictionary word.

**Reverse trie + rolling buffer:**

```js
class StreamChecker {
  constructor(words) {
    this.root = { children: new Map(), isEnd: false };
    for (const w of words) {
      let node = this.root;
      for (let i = w.length - 1; i >= 0; i--) {
        const ch = w[i];
        if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), isEnd: false });
        node = node.children.get(ch);
      }
      node.isEnd = true;
    }
    this.stream = [];
  }

  query(letter) {
    this.stream.push(letter);
    let node = this.root;
    for (let i = this.stream.length - 1; i >= 0; i--) {
      const ch = this.stream[i];
      if (!node.children.has(ch)) return false;
      node = node.children.get(ch);
      if (node.isEnd) return true;
    }
    return false;
  }
}
```

---

## 6. Search Suggestions (LC 1268)

Top 3 lexicographically smallest products matching prefix after each character typed.

**Pattern:** Trie insert products; after each keystroke DFS collect ≤3 in sorted order.

```js
function suggestedProducts(products, searchWord) {
  products.sort();
  const root = buildTrie(products);
  const result = [];
  let node = root;
  for (const ch of searchWord) {
    if (node && node.children.has(ch)) node = node.children.get(ch);
    else node = null;
    result.push(node ? collectTop3(node) : []);
  }
  return result;
}

function collectTop3(node) {
  const out = [];
  const dfs = (n, path) => {
    if (out.length >= 3) return;
    if (n.word) out.push(n.word);
    for (const ch of [...n.children.keys()].sort()) dfs(n.children.get(ch), path + ch);
  };
  dfs(node, "");
  return out;
}
```

---

## 7. Predictive Text Architecture

```
┌──────────────┐    ┌─────────────┐    ┌──────────────┐
│ User typing  │───►│ Prefix trie │───►│ Rank by freq │
└──────────────┘    └─────────────┘    └──────────────┘
                           │
                    Bigram trie (next word)
                           │
                    Personal dictionary (lazy delete)
```

**RN keyboard:** N-gram model + trie for current word; lazy-delete learned words on user correction.

---

## 8. Interview Q&A

### Q1: When lazy vs hard delete?

**A:** Lazy for high churn / simplicity. Hard when memory bounded (embedded, mobile). Compaction job for lazy.

### Q2: StreamChecker why reverse trie?

**A:** Match suffix of stream = prefix of reversed string — walk backward through buffer.

### Q3: Magic Dictionary brute force?

**A:** O(n × m) compare each word — trie with one mismatch branch is O(m × alphabet).

### Q4: Suggested products — why sort first?

**A:** DFS in sorted child order yields lex smallest 3 without sorting all matches.

### Q5: Predictive text beyond trie?

**A:** Add frequency weights at nodes; use heap for top-K; bigram trie for next-word prediction.

---

## Day 45 Cheat Sheet

```
Lazy delete:   isEnd = false
Hard delete:   post-order prune empty nodes
Suffix query:  reverse word trie
StreamChecker: reverse insert, walk stream backward
Magic dict:    DFS with one substitution allowed
Suggestions:   trie + collect 3 lex smallest
```

---

*End of Day 45 concepts*
