# Day 46 — Compressed Trie & Routing

**Week 7 — Tries** · **Topics:** Radix tree · Compressed trie · URL router · Deep linking · Lex order traversal

---

## Table of Contents

1. [Standard vs Compressed Trie](#1-standard-vs-compressed-trie)
2. [Radix Tree Structure](#2-radix-tree-structure)
3. [URL Router Simulation](#3-url-router-simulation)
4. [Trie II — Count & Prefix (LC 1804)](#4-trie-ii--count--prefix-lc-1804)
5. [Lexicographical Numbers (LC 386)](#5-lexicographical-numbers-lc-386)
6. [K-th Smallest in Lex Order (LC 440)](#6-k-th-smallest-in-lex-order-lc-440)
7. [Deep Link Router Pattern](#7-deep-link-router-pattern)
8. [Interview Q&A](#8-interview-qa)

---

## 1. Standard vs Compressed Trie

**Standard trie:** one node per character — simple but memory-heavy for sparse paths.

**Compressed trie (radix tree):** merge linear chains into single edge labeled with substring.

```
Standard:  r → o → u → t → e → r
Radix:     [root] --"router"--> (handler)
```

| | Standard | Radix |
|---|----------|-------|
| Memory | O(total chars) | O(number of nodes) fewer |
| Insert | O(m) | O(m) with split logic |
| Use case | Interviews, small dict | URL routers, IP tables, file paths |

---

## 2. Radix Tree Structure

```js
class RadixNode {
  constructor() {
    this.segment = "";      // edge label
    this.children = new Map(); // firstChar → RadixNode
    this.handler = null;    // route handler / isEnd
  }
}

class RadixTree {
  insert(path, handler) {
    this._insert(this.root, path, handler);
  }

  _insert(node, key, handler) {
    if (!key) { node.handler = handler; return; }

    const first = key[0];
    if (node.children.has(first)) {
      const child = node.children.get(first);
      const common = this._commonPrefix(child.segment, key);
      if (common.length < child.segment.length) {
        // Split node
        const split = new RadixNode();
        split.segment = child.segment.slice(common.length);
        split.children = child.children;
        split.handler = child.handler;
        child.segment = common;
        child.children = new Map([[split.segment[0], split]]);
        child.handler = null;
      }
      const remaining = key.slice(common.length + child.segment.length);
      if (remaining) this._insert(child, remaining, handler);
      else child.handler = handler;
    } else {
      const n = new RadixNode();
      n.segment = key;
      n.handler = handler;
      node.children.set(first, n);
    }
  }

  _commonPrefix(a, b) {
    let i = 0;
    while (i < a.length && i < b.length && a[i] === b[i]) i++;
    return a.slice(0, i);
  }

  match(path) {
    let node = this.root;
    let i = 0;
    while (i < path.length) {
      const child = node.children.get(path[i]);
      if (!child) return null;
      if (!path.startsWith(child.segment, i)) return null;
      i += child.segment.length;
      node = child;
    }
    return node.handler;
  }
}
```

---

## 3. URL Router Simulation

Express-style route matching with static and param segments:

```js
class RouteTrie {
  constructor() {
    this.root = { children: new Map(), handler: null, param: null };
  }

  addRoute(path, handler) {
    const parts = path.split("/").filter(Boolean);
    let node = this.root;
    for (const part of parts) {
      const key = part.startsWith(":") ? ":param" : part;
      if (!node.children.has(key)) {
        node.children.set(key, { children: new Map(), handler: null, param: null });
      }
      node = node.children.get(key);
      if (key === ":param") node.param = part.slice(1);
    }
    node.handler = handler;
  }

  match(url) {
    const parts = url.split("/").filter(Boolean);
    const params = {};
    return this._match(this.root, parts, 0, params);
  }

  _match(node, parts, i, params) {
    if (i === parts.length) return node.handler ? { handler: node.handler, params } : null;
    const part = parts[i];
    if (node.children.has(part)) {
      const r = this._match(node.children.get(part), parts, i + 1, params);
      if (r) return r;
    }
    if (node.children.has(":param")) {
      const child = node.children.get(":param");
      params[child.param] = part;
      const r = this._match(child, parts, i + 1, params);
      if (r) return r;
    }
    return null;
  }
}

const router = new RouteTrie();
router.addRoute("/users/:id", (p) => `User ${p.id}`);
router.addRoute("/users/:id/posts", (p) => `Posts of ${p.id}`);
router.match("/users/42"); // { handler, params: { id: "42" } }
```

---

## 4. Trie II — Count & Prefix (LC 1804)

Extended trie with `countWordsEqualTo`, `countWordsStartingWith`, `insert`, `erase`.

**Track `wordCount` and `prefixCount` at each node:**

```js
class TrieNodeII {
  constructor() {
    this.children = new Map();
    this.wordCount = 0;
    this.prefixCount = 0;
  }
}

class TrieII {
  insert(word) {
    let node = this.root;
    for (const ch of word) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNodeII());
      node = node.children.get(ch);
      node.prefixCount++;
    }
    node.wordCount++;
  }

  erase(word) {
    let node = this.root;
    for (const ch of word) {
      node = node.children.get(ch);
      node.prefixCount--;
    }
    node.wordCount--;
  }

  countWordsEqualTo(word) {
    const node = this._walk(word);
    return node ? node.wordCount : 0;
  }

  countWordsStartingWith(prefix) {
    const node = this._walk(prefix);
    return node ? node.prefixCount : 0;
  }
}
```

---

## 5. Lexicographical Numbers (LC 386)

Numbers 1..n in lex order = **DFS preorder on 10-ary trie** of digits.

```js
function lexicalOrder(n) {
  const result = [];
  const dfs = (cur) => {
    if (cur > n) return;
    result.push(cur);
    for (let d = 0; d <= 9; d++) {
      const next = cur === 0 ? 1 : cur * 10 + d;
      if (cur === 0 && d === 0) continue;
      dfs(next);
    }
  };
  dfs(0);
  return result;
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) excluding output |

---

## 6. K-th Smallest in Lex Order (LC 440)

Find k-th number in lex order without generating all — **count children in digit trie**.

```js
function findKthNumber(n, k) {
  let cur = 1;
  k--;
  while (k > 0) {
    const count = countSteps(n, cur, cur + 1);
    if (count <= k) {
      cur++;
      k -= count;
    } else {
      cur *= 10;
      k--;
    }
  }
  return cur;
}

function countSteps(n, first, next) {
  let steps = 0;
  while (first <= n) {
    steps += Math.min(n + 1, next) - first;
    first *= 10;
    next *= 10;
  }
  return steps;
}
```

---

## 7. Deep Link Router Pattern

**React Navigation linking config maps to trie:**

```js
const linking = {
  prefixes: ["myapp://", "https://app.example.com"],
  config: {
    screens: {
      Home: "",
      Profile: "users/:userId",
      Post: "users/:userId/posts/:postId",
    },
  },
};
```

Build trie from screen paths; match incoming URL segments; extract params — same as radix router.

```tsx
// RN deep link handler
function handleDeepLink(url: string) {
  const path = url.replace(/^myapp:\/\//, "");
  const match = deepLinkTrie.match(path);
  if (match) navigation.navigate(match.screen, match.params);
}
```

---

## 8. Interview Q&A

### Q1: When radix over standard trie?

**A:** Long shared prefixes (URLs, file paths), memory constraints. Standard trie simpler for interviews.

### Q2: URL router — trie vs regex?

**A:** Trie O(path length) deterministic; regex flexible but backtracking. Frameworks use trie-like structures.

### Q3: LC 386 vs sorting strings?

**A:** DFS on implicit trie O(n) vs sort O(n log n).

### Q4: LC 440 intuition?

**A:** Treat numbers as trie of digits; count subtree sizes to skip branches.

### Q5: Param routes order?

**A:** Static segments before `:param` — more specific routes first.

---

## Day 46 Cheat Sheet

```
Radix tree:   merge chains → substring edges
URL router:   segment trie + :param nodes
Trie II:      wordCount + prefixCount per node
Lex order:    DFS digit trie 1..n
K-th lex:     countSteps skip/advance
Deep links:   same trie match as web router
```

---

*End of Day 46 concepts*
