# Day 47 — Suffix Trie & Text Search

**Week 7 — Tries** · **Topics:** Suffix trie · Suffix tree · Text highlighter · Longest repeating/duplicate substring

---

## Table of Contents

1. [Prefix vs Suffix Trie](#1-prefix-vs-suffix-trie)
2. [Building a Suffix Trie](#2-building-a-suffix-trie)
3. [Text Search Highlighter](#3-text-search-highlighter)
4. [Prefix and Suffix Search (LC 745)](#4-prefix-and-suffix-search-lc-745)
5. [Longest Repeating Substring (LC 1062)](#5-longest-repeating-substring-lc-1062)
6. [Longest Duplicate Substring (LC 1044)](#6-longest-duplicate-substring-lc-1044)
7. [Syntax Highlighter Prefix Match](#7-syntax-highlighter-prefix-match)
8. [Interview Q&A](#8-interview-qa)

---

## 1. Prefix vs Suffix Trie

| | Prefix Trie | Suffix Trie |
|---|-------------|-------------|
| Stores | Words forward | All suffixes of text |
| Query | startsWith(word) | contains(substring)? |
| Space | O(total word chars) | O(n²) naive for length n |
| Alternative | — | Suffix array + LCP O(n) build |

**Key trick:** Insert all suffixes of `"banana"`:
`s[0..]`, `s[1..]`, … — or insert reversed string into prefix trie for suffix queries.

---

## 2. Building a Suffix Trie

### Naive — all suffixes

```js
class SuffixTrie {
  constructor(text) {
    this.root = { children: new Map(), indices: [] };
    for (let i = 0; i < text.length; i++) {
      this._insert(text.slice(i), i);
    }
  }

  _insert(suffix, startIndex) {
    let node = this.root;
    for (const ch of suffix) {
      if (!node.children.has(ch)) {
        node.children.set(ch, { children: new Map(), indices: [] });
      }
      node = node.children.get(ch);
      node.indices.push(startIndex);
    }
  }

  findAllOccurrences(pattern) {
    let node = this.root;
    for (const ch of pattern) {
      if (!node.children.has(ch)) return [];
      node = node.children.get(ch);
    }
    return node.indices;
  }
}

const st = new SuffixTrie("banana");
st.findAllOccurrences("ana"); // [1, 3]
```

### Optimized (interview mention)

- **Suffix array + binary search** — O(n log n) build
- **Rolling hash + binary search** (LC 1044) — O(n log n)
- **Ukkonen's algorithm** — O(n) true suffix tree (advanced)

---

## 3. Text Search Highlighter

Highlight all occurrences of query in document using suffix trie lookup:

```js
function highlightMatches(text, query) {
  if (!query) return [{ text, highlight: false }];
  const st = new SuffixTrie(text);
  const indices = st.findAllOccurrences(query);
  const ranges = indices.map((i) => [i, i + query.length]);

  // Merge overlapping ranges
  ranges.sort((a, b) => a[0] - b[0]);
  const merged = [];
  for (const [s, e] of ranges) {
    if (!merged.length || s > merged[merged.length - 1][1]) merged.push([s, e]);
    else merged[merged.length - 1][1] = Math.max(merged[merged.length - 1][1], e);
  }

  const parts = [];
  let prev = 0;
  for (const [s, e] of merged) {
    if (prev < s) parts.push({ text: text.slice(prev, s), highlight: false });
    parts.push({ text: text.slice(s, e), highlight: true });
    prev = e;
  }
  if (prev < text.length) parts.push({ text: text.slice(prev), highlight: false });
  return parts;
}
```

---

## 4. Prefix and Suffix Search (LC 745)

Design structure: `WordFilter(words)` + `f(prefix, suffix)` → largest index.

**Trick:** Insert `word + "#" + reversed_suffix` variants into trie, or store `{ prefix → { suffix → index } }`.

```js
class WordFilter {
  constructor(words) {
    this.map = new Map();
    for (let idx = 0; idx < words.length; idx++) {
      const w = words[idx];
      for (let preLen = 0; preLen <= w.length; preLen++) {
        for (let sufLen = 0; sufLen <= w.length; sufLen++) {
          const key = w.slice(0, preLen) + "#" + w.slice(w.length - sufLen);
          this.map.set(key, idx);
        }
      }
    }
  }

  f(prefix, suffix) {
    const key = prefix + "#" + suffix;
    return this.map.has(key) ? this.map.get(key) : -1;
  }
}
```

**Trie approach:** Insert padded strings `{suffix}#{word}#{prefix_padded}` for binary trie walk.

---

## 5. Longest Repeating Substring (LC 1062)

Longest substring that appears **at least twice** (can overlap).

### Binary search + hash

```js
function longestRepeatingSubstring(s) {
  const n = s.length;
  let lo = 0, hi = n - 1, ans = 0;

  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    if (hasDuplicate(s, mid)) { ans = mid; lo = mid + 1; }
    else hi = mid - 1;
  }
  return ans;
}

function hasDuplicate(s, len) {
  const seen = new Set();
  for (let i = 0; i <= s.length - len; i++) {
    const sub = s.slice(i, i + len);
    if (seen.has(sub)) return true;
    seen.add(sub);
  }
  return false;
}
```

### Rolling hash O(n log n)

```js
function hasDuplicateHash(s, len) {
  const base = 26, mod = 2n ** 61n - 1n;
  let h = 0n, pow = 1n;
  for (let i = 0; i < len; i++) {
    h = (h * base + BigInt(s.charCodeAt(i) - 97)) % mod;
    if (i > 0) pow = (pow * base) % mod;
  }
  const set = new Set([h.toString()]);
  for (let i = len; i < s.length; i++) {
    h = (h - BigInt(s.charCodeAt(i - len) - 97) * pow % mod + mod) % mod;
    h = (h * base + BigInt(s.charCodeAt(i) - 97)) % mod;
    const key = h.toString();
    if (set.has(key)) return true;
    set.add(key);
  }
  return false;
}
```

---

## 6. Longest Duplicate Substring (LC 1044)

Same as 1062 but return the substring; binary search + rolling hash with verification.

```js
function longestDupSubstring(s) {
  const n = s.length;
  let lo = 1, hi = n - 1, bestStart = 0, bestLen = 0;

  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    const start = findDupStart(s, mid);
    if (start >= 0) { bestStart = start; bestLen = mid; lo = mid + 1; }
    else hi = mid - 1;
  }
  return s.slice(bestStart, bestStart + bestLen);
}
```

**Suffix trie approach:** Deepest node with >1 index in subtree.

---

## 7. Syntax Highlighter Prefix Match

Tokenize by **longest prefix match** in keyword trie:

```js
const KEYWORDS = ["const", "let", "function", "return", "if", "else", "async", "await"];
const kwTrie = new Trie();
KEYWORDS.forEach((k) => kwTrie.insert(k));

function tokenize(code) {
  const tokens = [];
  let i = 0;
  while (i < code.length) {
    if (/\s/.test(code[i])) { i++; continue; }
    // Longest keyword prefix match
    let node = kwTrie.root, matched = "", j = i;
    while (j < code.length && node.children.has(code[j])) {
      node = node.children.get(code[j]);
      j++;
      if (node.isEnd) matched = code.slice(i, j);
    }
    if (matched) {
      tokens.push({ type: "keyword", value: matched });
      i += matched.length;
    } else {
      tokens.push({ type: "text", value: code[i] });
      i++;
    }
  }
  return tokens;
}
```

**RN syntax highlighter:** Same trie; render `Text` spans with color per token type.

---

## 8. Interview Q&A

### Q1: Suffix trie space?

**A:** Naive O(n²). Mention suffix array or rolling hash for interviews.

### Q2: LC 745 brute force?

**A:** O(n²) index pairs — trie/Map optimization for prefix+suffix combo.

### Q3: 1062 vs 1044?

**A:** Same binary search + hash; 1044 returns string, non-overlap not required (overlap OK).

### Q4: Highlighter performance?

**A:** Suffix array for static doc; re-search on query change only walks trie O(m + occ).

### Q5: Aho-Corasick vs suffix trie?

**A:** Aho-Corasick multi-pattern; suffix trie single text all suffixes.

---

## Day 47 Cheat Sheet

```
Suffix trie:     insert all s[i..]
Highlight:       find indices → merge ranges
LC 745:          prefix#suffix index map
LC 1062/1044:    binary search length + rolling hash
Syntax HL:       longest prefix match in keyword trie
```

---

*End of Day 47 concepts*
