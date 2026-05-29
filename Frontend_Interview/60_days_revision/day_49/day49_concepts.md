# Day 49 — Week 7 Revision (Tries)

**Week 7 — Tries (Sun)** · **Topics:** Full trie revision · Spell checker + edit distance · Voice command prefix · Timed mock

---

## Table of Contents

1. [Week 7 Knowledge Map](#1-week-7-knowledge-map)
2. [Trie Pattern Decision Tree](#2-trie-pattern-decision-tree)
3. [Spell Checker + Edit Distance](#3-spell-checker--edit-distance)
4. [Voice Command Prefix Matching](#4-voice-command-prefix-matching)
5. [Day-by-Day Recap](#5-day-by-day-recap)
6. [Complexity Reference](#6-complexity-reference)
7. [Interview Q&A Marathon](#7-interview-qa-marathon)
8. [Mock Preview (LC 208, 211, 212)](#8-mock-preview-lc-208-211-212)

---

## 1. Week 7 Knowledge Map

```
                    TRIE FAMILY
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   Standard          Compressed       Binary
   (Map/Array)       (Radix/URL)      (XOR/IP)
        │                │                │
   208 211 212      1804 386 440      421 1707
   648 677 720      Router/DeepLink   1938 CIDR
   1268 1032 676
        │
   Suffix/Reverse
   745 1062 1044
```

---

## 2. Trie Pattern Decision Tree

| Problem signal | Pattern | Example |
|----------------|---------|---------|
| Prefix search / autocomplete | Standard trie + DFS | 208, 1268 |
| Wildcard `.` in word | Trie + DFS branch | 211 |
| Grid + multiple words | Board DFS + trie prune | 212 |
| Shortest root replacement | Walk until first isEnd | 648 |
| Prefix sum of keys | Delta count on path | 677 |
| All prefixes valid | BFS/DFS valid path | 720 |
| Stream suffix match | Reverse trie | 1032 |
| One char substitution | DFS one mismatch | 676 |
| Count words / erase | wordCount + prefixCount | 1804 |
| Lex order 1..n | Digit trie DFS | 386, 440 |
| Substring duplicate | Binary search + hash | 1062, 1044 |
| Prefix + suffix index | Padded key map | 745 |
| Max XOR | Binary trie opposite bit | 421 |
| Range + XOR | Offline sort + trie | 1707 |
| Subtree XOR query | Tree DFS + merge trie | 1938 |

---

## 3. Spell Checker + Edit Distance

Combine **trie** (fast prefix candidates) with **Levenshtein** (typo tolerance):

```js
function levenshtein(a, b) {
  const dp = Array.from({ length: a.length + 1 }, (_, i) =>
    Array.from({ length: b.length + 1 }, (_, j) => (i === 0 ? j : j === 0 ? i : 0))
  );
  for (let i = 1; i <= a.length; i++) {
    for (let j = 1; j <= b.length; j++) {
      dp[i][j] = a[i - 1] === b[j - 1]
        ? dp[i - 1][j - 1]
        : 1 + Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
    }
  }
  return dp[a.length][b.length];
}

class SpellChecker {
  constructor(dictionary) {
    this.trie = new Trie();
    dictionary.forEach((w) => this.trie.insert(w));
    this.words = dictionary;
  }

  suggest(word, maxDist = 2) {
    // 1. Exact match
    if (this.trie.search(word)) return [word];

    // 2. Prefix narrow — same first 2 chars
    const prefix = word.slice(0, 2);
    const candidates = this.trie.startsWith(prefix)
      ? this.trie.suggest(prefix, 50)
      : this.words;

    // 3. Edit distance filter + rank
    return candidates
      .map((w) => ({ w, d: levenshtein(word, w) }))
      .filter(({ d }) => d <= maxDist)
      .sort((a, b) => a.d - b.d || a.w.localeCompare(b.w))
      .slice(0, 5)
      .map(({ w }) => w);
  }
}
```

**Optimization:** BK-tree or trie-guided DP for production spell check.

---

## 4. Voice Command Prefix Matching

Voice input is fuzzy — match **spoken prefix** against command trie:

```js
const COMMANDS = [
  "open settings",
  "open camera",
  "play music",
  "pause music",
  "send message",
  "set alarm",
];

class VoiceCommandTrie {
  constructor(commands) {
    this.root = { children: new Map(), command: null };
    for (const cmd of commands) {
      let node = this.root;
      for (const ch of cmd) {
        if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), command: null });
        node = node.children.get(ch);
      }
      node.command = cmd;
    }
  }

  match(transcript) {
    const t = transcript.toLowerCase().trim();
    // Exact prefix walk
    let node = this.root;
    for (const ch of t) {
      if (!node.children.has(ch)) break;
      node = node.children.get(ch);
      if (node.command && node.command.startsWith(t)) return node.command;
    }
    // Fuzzy: commands starting with transcript prefix
    return this._collectCommands(t)[0] ?? null;
  }

  _collectCommands(prefix) {
    let node = this.root;
    for (const ch of prefix) {
      if (!node.children.has(ch)) return [];
      node = node.children.get(ch);
    }
    const out = [];
    const dfs = (n) => {
      if (n.command) out.push(n.command);
      for (const c of n.children.values()) dfs(c);
    };
    dfs(node);
    return out.sort();
  }
}
```

**RN:** `@react-native-voice/voice` → transcript → trie match → navigate.

---

## 5. Day-by-Day Recap

| Day | Core | LC |
|-----|------|-----|
| **43** | Node Map/Array, autocomplete | 208, 211, 212 |
| **44** | insert/search/startsWith, dict | 648, 677, 720 |
| **45** | Lazy/hard delete, stream | 1268, 1032, 676 |
| **46** | Radix, URL router, lex order | 1804, 440, 386 |
| **47** | Suffix trie, highlighter | 745, 1062, 1044 |
| **48** | Binary trie, XOR, IP | 421, 1707, 1938 |
| **49** | Revision + mock | 208, 211, 212 timed |

---

## 6. Complexity Reference

| Operation | Time | Notes |
|-----------|------|-------|
| Trie insert/search | O(m) | m = word length |
| Autocomplete k results | O(m + k) | DFS |
| Wildcard search | O(26^d × m) | d = dot count |
| Board + trie | O(R×C×4^L) | pruned |
| Binary trie XOR | O(32) | per query |
| Suffix trie naive build | O(n²) | mention better |
| Edit distance | O(a × b) | spell check |

---

## 7. Interview Q&A Marathon

### Q1: Implement trie from scratch in 5 min?

**A:** Map children, insert/walk/search/startsWith — 15 lines.

### Q2: When NOT to use trie?

**A:** Exact lookups only, no prefix — HashMap O(1). Small static set — sorted array.

### Q3: Memory optimization?

**A:** Radix compression, array for a-z, lazy delete with compaction.

### Q4: Trie thread-safe?

**A:** Read-copy-update, immutable snapshots, or lock per subtree.

### Q5: Frontend real-world uses?

**A:** Autocomplete, routers, spell check, command palette, IP/CDN routing, code completion.

### Q6: 212 vs 79 (Word Search I)?

**A:** 212 needs trie to prune multiple words simultaneously; 79 single word.

### Q7: Reverse trie use cases?

**A:** StreamChecker suffix, suffix queries, autocomplete on reversed corpus.

### Q8: Binary trie vs bit manipulation?

**A:** Trie for dynamic set max XOR; bit tricks for fixed formulas.

---

## 8. Mock Preview (LC 208, 211, 212)

Today's LeetCode file is a **45-minute timed mock** covering Day 43 fundamentals:

| # | Problem | Target |
|---|---------|--------|
| 208 | Implement Trie | 12 min |
| 211 | Add Search Word | 15 min |
| 212 | Word Search II | 18 min |

**Before mock:** Write trie node on paper. State wildcard DFS and board prune aloud.

---

## Day 49 Cheat Sheet

```
Week 7 mastered:
  Standard trie → wildcard → board prune
  Dict ops → delete → stream → suggestions
  Radix router → suffix → binary XOR
Spell check: trie prefix + Levenshtein
Voice: transcript prefix → command trie
Mock: 208, 211, 212 @ 45 min
```

---

*End of Day 49 concepts — Week 7 complete*
