# Day 43 — Trie Fundamentals

**Week 7 — Tries** · **Topics:** Trie node structure · Array vs Map children · Autocomplete · Prefix tree basics

---

## Table of Contents

1. [What Is a Trie?](#1-what-is-a-trie)
2. [Node Structure — Array vs Map](#2-node-structure--array-vs-map)
3. [Core Operations](#3-core-operations)
4. [Autocomplete Pattern](#4-autocomplete-pattern)
5. [Complexity & Trade-offs](#5-complexity--trade-offs)
6. [Trie vs HashMap vs BST](#6-trie-vs-hashmap-vs-bst)
7. [Interview Q&A](#7-interview-qa)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. What Is a Trie?

A **Trie** (prefix tree) stores strings character-by-character. Each path from root to a marked node represents one word.

```
        root
       / | \
      a  b  c
      |     |
     p*    a
      |     |
     p*    t*
      |
     l*
     e*

* = isEnd (word ends here)
Words: app, apple, cat
```

**Why frontend cares:** search suggestions, spell check, route matching, typeahead, command palettes.

---

## 2. Node Structure — Array vs Map

### Array children (a–z, fixed alphabet)

Best when alphabet is small and known (lowercase English).

```js
class TrieNode {
  constructor() {
    this.children = new Array(26).fill(null);
    this.isEnd = false;
  }

  index(ch) {
    return ch.charCodeAt(0) - "a".charCodeAt(0);
  }
}
```

| Pros | Cons |
|------|------|
| O(1) child lookup | Wastes space for sparse branches |
| Cache-friendly | Fixed to 26 letters |
| Fast in interviews | Unicode needs different sizing |

### Map children (dynamic alphabet)

Best for mixed case, symbols, Unicode, URLs.

```js
class TrieNode {
  constructor() {
    this.children = new Map(); // char → TrieNode
    this.isEnd = false;
  }
}
```

| Pros | Cons |
|------|------|
| Sparse — only stores used chars | Map overhead per node |
| Any character set | Slightly slower than array |
| Production default | |

### Hybrid (interview sweet spot)

Use **Map** in JS interviews unless problem says lowercase a–z only — then array is fine and shows you know both.

```js
class TrieNode {
  constructor() {
    this.children = new Map();
    this.isEnd = false;
    this.word = null; // optional: store full word at terminal node (LC 212)
  }
}
```

---

## 3. Core Operations

### Insert

```js
class Trie {
  constructor() {
    this.root = new TrieNode();
  }

  insert(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) {
        node.children.set(ch, new TrieNode());
      }
      node = node.children.get(ch);
    }
    node.isEnd = true;
    node.word = word; // useful for Word Search II
  }
}
```

### Search (exact match)

```js
  search(word) {
    const node = this._walk(word);
    return node !== null && node.isEnd;
  }

  _walk(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) return null;
      node = node.children.get(ch);
    }
    return node;
  }
```

### startsWith (prefix match)

```js
  startsWith(prefix) {
    return this._walk(prefix) !== null;
  }
```

---

## 4. Autocomplete Pattern

Given prefix `pre`, DFS/BFS from prefix node to collect all words.

```js
  suggest(prefix, limit = 5) {
    const node = this._walk(prefix);
    if (!node) return [];

    const results = [];
    const dfs = (n, path) => {
      if (results.length >= limit) return;
      if (n.isEnd) results.push(path);
      for (const [ch, child] of n.children) {
        dfs(child, path + ch);
      }
    };
    dfs(node, prefix);
    return results;
  }
```

### Frontend autocomplete flow

```
User types "ja"
  → debounce 150ms
  → trie.startsWith("ja") ? suggest("ja", 5)
  → render dropdown with keyboard nav
```

**Optimization:** Store `count` or `rank` at each node for top-K without full DFS.

---

## 5. Complexity & Trade-offs

| Operation | Time | Space (total) |
|-----------|------|---------------|
| Insert word length m | O(m) | O(total chars stored) |
| Search | O(m) | — |
| startsWith | O(m) | — |
| Autocomplete all matches | O(m + k) | k = output size |

**Space:** Worst case O(n × m) for n words avg length m. Shared prefixes collapse — `"app"`, `"apple"` share `"app"` path.

---

## 6. Trie vs HashMap vs BST

| Structure | Prefix queries | Exact lookup | Space |
|-----------|----------------|--------------|-------|
| **Trie** | O(m) | O(m) | Shared prefixes |
| **HashMap** | O(n) scan all keys | O(1) avg | No sharing |
| **BST** | O(n) inorder filter | O(log n) avg | No prefix native |

**When to pick Trie:** prefix search, autocomplete, dictionary, wildcard search (Day 43 LC 211).

---

## 7. Interview Q&A

### Q1: Array or Map for children?

**A:** Lowercase-only fixed alphabet → array O(1). General strings / URLs → Map. Mention both; default to Map in JS.

### Q2: How does autocomplete scale to millions of terms?

**A:** Trie for prefix lookup + heap for top-K by popularity. Cache hot prefixes. Limit DFS depth and result count.

### Q3: Case sensitivity?

**A:** Normalize to lowercase on insert/search, or store both variants. Document choice.

### Q4: What if two words share a prefix?

**A:** Shared path until divergence. Each terminal node marks `isEnd`; intermediate nodes can also be terminal (`"a"` and `"app"`).

### Q5: Trie vs storing sorted array + binary search?

**A:** Sorted array + binary search finds prefix range in O(log n + k). Trie is O(m + k) — better when m (prefix length) << log n and updates are frequent.

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| Node structure | [§2](#2-node-structure--array-vs-map) |
| Insert/search/startsWith | [§3](#3-core-operations) |
| Autocomplete DFS | [§4](#4-autocomplete-pattern) |
| Complexity | [§5](#5-complexity--trade-offs) |

---

## Day 43 Cheat Sheet

```
TrieNode: children (Map|Array) + isEnd (+ optional word/rank)
insert:   walk + create nodes + mark isEnd
search:   walk + check isEnd
startsWith: walk only
autocomplete: DFS from prefix node
Use Map in JS unless a-z only
```

---

*End of Day 43 concepts*
