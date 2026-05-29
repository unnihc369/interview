# Day 44 — Trie Dictionary Operations

**Week 7 — Tries** · **Topics:** insert · search · startsWith · Dictionary prefix search · Map sum on trie

---

## Table of Contents

1. [The Three Core APIs](#1-the-three-core-apis)
2. [Dictionary Prefix Search](#2-dictionary-prefix-search)
3. [Prefix Replacement (LC 648)](#3-prefix-replacement-lc-648)
4. [Map Sum on Trie (LC 677)](#4-map-sum-on-trie-lc-677)
5. [Longest Word in Dictionary (LC 720)](#5-longest-word-in-dictionary-lc-720)
6. [Offline Dictionary Architecture](#6-offline-dictionary-architecture)
7. [Interview Q&A](#7-interview-qa)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. The Three Core APIs

Every trie interview starts here:

| Method | Returns true when | Typical use |
|--------|-------------------|-------------|
| `insert(word)` | — | Build structure |
| `search(word)` | Exact word exists | Spell check lookup |
| `startsWith(prefix)` | Any word has prefix | Autocomplete gate |

```js
class DictionaryTrie {
  constructor() {
    this.root = { children: new Map(), isEnd: false };
  }

  insert(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), isEnd: false });
      node = node.children.get(ch);
    }
    node.isEnd = true;
  }

  search(word) {
    const node = this._walk(word);
    return !!node && node.isEnd;
  }

  startsWith(prefix) {
    return this._walk(prefix) !== null;
  }

  _walk(s) {
    let node = this.root;
    for (const ch of s) {
      if (!node.children.has(ch)) return null;
      node = node.children.get(ch);
    }
    return node;
  }
}
```

**Invariant:** `startsWith("app")` true does NOT imply `search("app")` true unless `"app"` was inserted as a complete word.

---

## 2. Dictionary Prefix Search

### Collect all words with prefix

```js
  findAllWithPrefix(prefix) {
    const node = this._walk(prefix);
    if (!node) return [];
    const words = [];
    const dfs = (n, path) => {
      if (n.isEnd) words.push(path);
      for (const [ch, child] of n.children) dfs(child, path + ch);
    };
    dfs(node, prefix);
    return words;
  }
```

### Count words with prefix

Store `count` at each node during insert; decrement on delete (Day 45).

```js
  insertWithCount(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) {
        node.children.set(ch, { children: new Map(), isEnd: false, count: 0 });
      }
      node = node.children.get(ch);
      node.count = (node.count || 0) + 1;
    }
    node.isEnd = true;
  }

  countPrefix(prefix) {
    const node = this._walk(prefix);
    return node ? node.count || 0 : 0;
  }
```

---

## 3. Prefix Replacement (LC 648)

**Problem:** Replace words with shortest dictionary root prefix.

```
dictionary = ["cat","bat","rat"]
sentence = "the cattle was rattled by the battery"
→ "the cat was rat by the bat"
```

**Algorithm:** For each word, walk trie until first `isEnd` — that's the replacement.

```js
function replaceWords(dictionary, sentence) {
  const root = new DictionaryTrie();
  dictionary.forEach((w) => root.insert(w));

  return sentence.split(" ").map((word) => {
    let node = root.root;
    let i = 0;
    while (i < word.length && node.children.has(word[i])) {
      node = node.children.get(word[i]);
      i++;
      if (node.isEnd) return word.slice(0, i);
    }
    return word;
  }).join(" ");
}
```

| Time | Space |
|------|-------|
| O(d + s × m) | O(d) dict size |

---

## 4. Map Sum on Trie (LC 677)

Store **value at terminal node**; propagate sum up OR store cumulative at each node.

```js
class MapSum {
  constructor() {
    this.root = { children: new Map(), val: 0 };
    this.keys = new Map(); // key → last value
  }

  insert(key, val) {
    const delta = val - (this.keys.get(key) ?? 0);
    this.keys.set(key, val);
    let node = this.root;
    node.val += delta;
    for (const ch of key) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), val: 0 });
      node = node.children.get(ch);
      node.val += delta;
    }
  }

  sum(prefix) {
    let node = this.root;
    for (const ch of prefix) {
      if (!node.children.has(ch)) return 0;
      node = node.children.get(ch);
    }
    return node.val;
  }
}
```

**Key insight:** On update, subtract old value, add new — only walk key length once.

---

## 5. Longest Word in Dictionary (LC 720)

Word is valid if **every prefix** is also in dictionary.

**BFS from root** (lex order) or DFS checking all prefixes exist:

```js
function longestWord(words) {
  const trie = new DictionaryTrie();
  words.sort().forEach((w) => trie.insert(w));

  let best = "";
  const dfs = (node, path) => {
    if (node.isEnd && path.length > best.length) best = path;
    for (const [ch, child] of [...node.children.entries()].sort()) {
      if (child.isEnd) dfs(child, path + ch);
    }
  };
  dfs(trie.root, "");
  return best;
}
```

Only traverse branches where every node on path is terminal (except root).

---

## 6. Offline Dictionary Architecture

**RN / PWA offline dictionary pattern:**

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│ JSON/word   │ ──► │ Build trie   │ ──► │ IndexedDB / │
│ list bundle │     │ on first run │     │ AsyncStorage│
└─────────────┘     └──────────────┘     └─────────────┘
                            │
                            ▼
                   prefix search O(m)
                   no network required
```

```js
// Hydrate from bundled JSON
async function loadOfflineDict() {
  const cached = await AsyncStorage.getItem("dict:trie");
  if (cached) return JSON.parse(cached);
  const words = require("./words.json");
  const trie = new DictionaryTrie();
  words.forEach((w) => trie.insert(w.definition ? w.word : w));
  await AsyncStorage.setItem("dict:trie", JSON.stringify(serializeTrie(trie)));
  return trie;
}
```

---

## 7. Interview Q&A

### Q1: Difference between search and startsWith?

**A:** `search` needs `isEnd === true` at end of walk. `startsWith` only checks path exists — `"app"` prefix of `"apple"` even if `"app"` not a word.

### Q2: How to implement prefix frequency?

**A:** Increment `count` on every node along insert path. Prefix count = node at prefix end `.count`.

### Q3: Replace Words — why trie not hash set?

**A:** Need **shortest** matching prefix per word — trie stops at first `isEnd` while scanning left to right.

### Q4: MapSum update same key?

**A:** Track previous value in Map; apply delta to all nodes on path.

### Q5: Longest word — why sort words?

**A:** Lex tie-break: BFS/DFS in sorted child order ensures `"w"` beats `"wa"` when same length rules apply.

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| insert/search/startsWith | [§1](#1-the-three-core-apis) |
| Prefix enumeration | [§2](#2-dictionary-prefix-search) |
| Replace Words | [§3](#3-prefix-replacement-lc-648) |
| MapSum | [§4](#4-map-sum-on-trie-lc-677) |
| Offline dict | [§6](#6-offline-dictionary-architecture) |

---

## Day 44 Cheat Sheet

```
search      → walk + isEnd
startsWith  → walk only
replaceWords → walk until first isEnd
MapSum      → delta update on path
longestWord → all prefixes must be words
Offline     → bundle JSON → trie → persist
```

---

*End of Day 44 concepts*
