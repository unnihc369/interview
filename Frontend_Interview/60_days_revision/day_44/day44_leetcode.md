# Day 44 — LeetCode (Dictionary Trie)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 648 | [Replace Words](#1-replace-words-648) | Medium | Shortest prefix root |
| 677 | [Map Sum Pairs](#2-map-sum-pairs-677) | Medium | Trie + prefix sum |
| 720 | [Longest Word in Dictionary](#3-longest-word-in-dictionary-720) | Medium | Buildable word trie |

---

## Table of Contents

1. [Replace Words (648)](#1-replace-words-648)
2. [Map Sum Pairs (677)](#2-map-sum-pairs-677)
3. [Longest Word in Dictionary (720)](#3-longest-word-in-dictionary-720)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Replace Words (648)

### Problem (short)

Replace each word in sentence with shortest **dictionary root** prefix if exists.

```
dictionary = ["cat","bat","rat"]
sentence = "the cattle was rattled by the battery"
Output: "the cat was rat by the bat"
```

### Hint 1 — Trie dictionary

Insert all roots. For each word, walk until first `isEnd`.

### Hint 2 — Don't over-replace

Stop at **first** terminal node — shortest root wins.

### Solution — JavaScript

```js
/**
 * @param {string[]} dictionary
 * @param {string} sentence
 * @return {string}
 */
function replaceWords(dictionary, sentence) {
  const root = { children: new Map(), isEnd: false };
  for (const w of dictionary) {
    let node = root;
    for (const ch of w) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), isEnd: false });
      node = node.children.get(ch);
    }
    node.isEnd = true;
  }

  const replace = (word) => {
    let node = root;
    for (let i = 0; i < word.length; i++) {
      if (!node.children.has(word[i])) return word;
      node = node.children.get(word[i]);
      if (node.isEnd) return word.slice(0, i + 1);
    }
    return word;
  };

  return sentence.split(" ").map(replace).join(" ");
}
```

| Time | Space |
|------|-------|
| O(d + w × m) | O(d) |

---

## 2. Map Sum Pairs (677)

### Problem (short)

Implement `MapSum` with `insert(key, val)` and `sum(prefix)`.

```
insert("apple", 3)
sum("ap") → 3
insert("app", 2)
sum("ap") → 5
```

### Hint 1 — Value at nodes

Each node stores cumulative sum of all keys passing through.

### Hint 2 — Update delta

When key exists, add `(newVal - oldVal)` to path nodes only.

### Solution — JavaScript

```js
class MapSum {
  constructor() {
    this.root = { children: new Map(), val: 0 };
    this.map = new Map();
  }

  insert(key, val) {
    const delta = val - (this.map.get(key) ?? 0);
    this.map.set(key, val);
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

| insert | sum | Space |
|--------|-----|-------|
| O(m) | O(m) | O(n × m) |

---

## 3. Longest Word in Dictionary (720)

### Problem (short)

Return longest word where every prefix is also in dictionary. Lex smallest on tie.

```
words = ["w","wo","wor","worl","world"]
Output: "world"
```

### Hint 1 — Trie + valid path

Word valid iff every char along path hits `isEnd` (except empty root).

### Hint 2 — BFS lex order

Sort children; BFS from root — last valid terminal = answer. Or DFS tracking best.

### Solution — JavaScript (DFS)

```js
/**
 * @param {string[]} words
 * @return {string}
 */
function longestWord(words) {
  const root = { children: new Map(), isEnd: false };
  for (const w of words) {
    let node = root;
    for (const ch of w) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), isEnd: false });
      node = node.children.get(ch);
    }
    node.isEnd = true;
  }

  let best = "";
  const dfs = (node, path) => {
    if (path && node.isEnd) {
      if (path.length > best.length || (path.length === best.length && path < best)) {
        best = path;
      }
    }
    for (const ch of [...node.children.keys()].sort()) {
      const child = node.children.get(ch);
      if (child.isEnd || path === "") dfs(child, path + ch);
    }
  };

  // Only extend paths where every prefix is a word
  const dfsValid = (node, path) => {
    if (node.isEnd && path.length > best.length) best = path;
    if (node.isEnd && path.length === best.length && path < best) best = path;
    for (const ch of [...node.children.keys()].sort()) {
      const child = node.children.get(ch);
      if (node.isEnd || path === "") dfsValid(child, path + ch);
    }
  };

  dfsValid(root, "");
  return best;
}
```

### Cleaner BFS approach

```js
function longestWordBFS(words) {
  words.sort();
  const set = new Set(words);
  const queue = [""];
  let ans = "";
  while (queue.length) {
    const w = queue.shift();
    if (w.length > ans.length) ans = w;
    for (const word of words) {
      if (word.startsWith(w) && word.length === w.length + 1 && set.has(w)) {
        queue.push(word);
      }
    }
  }
  return ans;
}
```

| Time | Space |
|------|-------|
| O(n × m) | O(n × m) |

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **648** | Trie + first isEnd | Shortest root replacement |
| **677** | Prefix cumulative sum | Delta on update |
| **720** | All-prefixes-valid | BFS/DFS lex order |

### Follow-ups

| Question | Answer |
|----------|--------|
| 648 hash set? | Can't find shortest prefix efficiently |
| 677 delete key? | Subtract delta on path |
| 720 tie-break | Lex smallest when same length |

### Test cases

```js
replaceWords(["cat","bat","rat"], "the cattle was rattled by the battery");
// "the cat was rat by the bat"

const ms = new MapSum();
ms.insert("apple", 3);
console.assert(ms.sum("ap") === 3);
ms.insert("app", 2);
console.assert(ms.sum("ap") === 5);

console.assert(longestWord(["w","wo","wor","worl","world"]) === "world");
```

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 648 | O(d + w×m) | O(d) |
| 677 | O(m) per op | O(keys) |
| 720 | O(n log n + n×m) | O(n×m) |

---

*End of Day 44 LeetCode*
