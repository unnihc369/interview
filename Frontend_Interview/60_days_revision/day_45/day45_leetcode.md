# Day 45 — LeetCode (Advanced Trie)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 1268 | [Search Suggestions System](#1-search-suggestions-system-1268) | Medium | Trie + top 3 lex |
| 1032 | [Stream of Characters](#2-stream-of-characters-1032) | Hard | Reverse trie stream |
| 676 | [Implement Magic Dictionary](#3-implement-magic-dictionary-676) | Medium | Trie one mismatch |

---

## Table of Contents

1. [Search Suggestions System (1268)](#1-search-suggestions-system-1268)
2. [Stream of Characters (1032)](#2-stream-of-characters-1032)
3. [Implement Magic Dictionary (676)](#3-implement-magic-dictionary-676)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Search Suggestions System (1268)

### Problem (short)

After each character of `searchWord`, return top 3 lexicographically smallest matching products.

```
products = ["mobile","mouse","moneypot","monitor","mousepad"]
searchWord = "mouse"
Output: [["mobile","moneypot","monitor"],["mobile","moneypot","monitor"],["mouse","mousepad"],["mouse","mousepad"],["mouse","mousepad"]]
```

### Hint 1 — Sort + trie

Sort products first. Insert into trie with word reference at terminal.

### Hint 2 — Per keystroke

Walk trie; if broken return `[]`. Else DFS collect first 3 in lex order.

### Solution — JavaScript

```js
/**
 * @param {string[]} products
 * @param {string} searchWord
 * @return {string[][]}
 */
function suggestedProducts(products, searchWord) {
  products.sort();
  const root = { children: new Map(), word: null };
  for (const p of products) {
    let node = root;
    for (const ch of p) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), word: null });
      node = node.children.get(ch);
    }
    node.word = p;
  }

  const top3 = (node) => {
    const res = [];
    const dfs = (n) => {
      if (res.length >= 3) return;
      if (n.word) res.push(n.word);
      for (const ch of [...n.children.keys()].sort()) dfs(n.children.get(ch));
    };
    dfs(node);
    return res;
  };

  const result = [];
  let node = root;
  for (const ch of searchWord) {
    if (node && node.children.has(ch)) node = node.children.get(ch);
    else node = null;
    result.push(node ? top3(node) : []);
  }
  return result;
}
```

| Time | Space |
|------|-------|
| O(n log n + m + k) | O(n × m) |

---

## 2. Stream of Characters (1032)

### Problem (short)

Given dictionary, process stream one char at a time. `query(letter)` returns true if any suffix of stream matches a dictionary word.

### Hint 1 — Reverse words in trie

Insert `"abc"` as `"cba"`. Walk stream backward.

### Hint 2 — Buffer limit

Max word length bounds backward walk — O(maxLen) per query.

### Solution — JavaScript

```js
class StreamChecker {
  constructor(words) {
    this.root = { children: new Map(), isEnd: false };
    this.maxLen = 0;
    for (const w of words) {
      this.maxLen = Math.max(this.maxLen, w.length);
      let node = this.root;
      for (let i = w.length - 1; i >= 0; i--) {
        const ch = w[i];
        if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), isEnd: false });
        node = node.children.get(ch);
      }
      node.isEnd = true;
    }
    this.buf = [];
  }

  query(letter) {
    this.buf.push(letter);
    if (this.buf.length > this.maxLen) this.buf.shift();

    let node = this.root;
    for (let i = this.buf.length - 1; i >= 0; i--) {
      const ch = this.buf[i];
      if (!node.children.has(ch)) return false;
      node = node.children.get(ch);
      if (node.isEnd) return true;
    }
    return false;
  }
}
```

| query | Space |
|-------|-------|
| O(maxLen) | O(total chars) |

---

## 3. Implement Magic Dictionary (676)

### Problem (short)

Build dictionary. `search(word)` true if exactly one edit (change one char) matches a dict word.

### Hint 1 — Brute O(n×m)

Compare each dict word — acceptable if mentioned then optimize.

### Hint 2 — Trie DFS

Allow one mismatch during walk; at end require exactly one used.

### Solution — JavaScript

```js
class MagicDictionary {
  constructor() {
    this.root = { children: new Map(), isEnd: false };
  }

  buildDict(dictionary) {
    for (const w of dictionary) {
      let node = this.root;
      for (const ch of w) {
        if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), isEnd: false });
        node = node.children.get(ch);
      }
      node.isEnd = true;
    }
  }

  search(word) {
    return this._dfs(this.root, word, 0, false);
  }

  _dfs(node, word, i, mismatched) {
    if (i === word.length) return mismatched && node.isEnd;

    const ch = word[i];
    if (node.children.has(ch)) {
      if (this._dfs(node.children.get(ch), word, i + 1, mismatched)) return true;
    }
    if (mismatched) return false;

    for (const [c, child] of node.children) {
      if (c !== ch && this._dfs(child, word, i + 1, true)) return true;
    }
    return false;
  }
}
```

| buildDict | search | Space |
|-----------|--------|-------|
| O(n × m) | O(m × α) | O(n × m) |

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **1268** | Sorted trie + top 3 DFS | Per-char suggestion list |
| **1032** | Reverse trie + buffer | Suffix match stream |
| **676** | Trie + one mismatch | Magic search DFS |

### Follow-ups

| Question | Answer |
|----------|--------|
| 1268 without trie? | Sort + binary search prefix per query |
| 1032 Aho-Corasick? | Overkill for suffix-only; reverse trie suffices |
| 676 add/delete? | Standard trie ops + magic search |

### Test cases

```js
suggestedProducts(
  ["mobile","mouse","moneypot","monitor","mousepad"],
  "mouse"
); // 5 arrays, last three ["mouse","mousepad"]

const sc = new StreamChecker(["cd","f","kl"]);
console.assert(sc.query("a") === false);
console.assert(sc.query("b") === false);

const md = new MagicDictionary();
md.buildDict(["hello","leetcode"]);
console.assert(md.search("hello") === false);
console.assert(md.search("hhllo") === true);
```

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 1268 | O(n log n + m) | O(n×m) |
| 1032 | O(L) per query | O(n×m) |
| 676 | O(m×α) search | O(n×m) |

---

*End of Day 45 LeetCode*
