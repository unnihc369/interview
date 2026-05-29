# Day 48 — Bitwise Trie

**Week 7 — Tries** · **Topics:** Binary trie · XOR maximization · IP routing · Bit manipulation · Range queries

---

## Table of Contents

1. [Binary Trie Structure](#1-binary-trie-structure)
2. [Maximum XOR (LC 421)](#2-maximum-xor-lc-421)
3. [IP Routing Table](#3-ip-routing-table)
4. [Range Bitwise AND (LC 1707)](#4-range-bitwise-and-lc-1707)
5. [Max Genetic Score (LC 1938)](#5-max-genetic-score-lc-1938)
6. [Network Prefix Match (RN)](#6-network-prefix-match-rn)
7. [Interview Q&A](#7-interview-qa)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. Binary Trie Structure

Each bit (0 or 1) is a branch — 32 levels for 32-bit integers.

```js
class BinaryTrieNode {
  constructor() {
    this.zero = null;
    this.one = null;
    this.count = 0; // nodes in subtree (for deletion/routing)
  }
}

class BinaryTrie {
  insert(num, bits = 31) {
    let node = this.root;
    for (let i = bits; i >= 0; i--) {
      const bit = (num >> i) & 1;
      if (bit === 0) {
        if (!node.zero) node.zero = new BinaryTrieNode();
        node = node.zero;
      } else {
        if (!node.one) node.one = new BinaryTrieNode();
        node = node.one;
      }
      node.count++;
    }
  }

  constructor() {
    this.root = new BinaryTrieNode();
  }
}
```

```
Insert 5 (101), 3 (011):

        root
       /    \
      0      1
      |      |
     ...    ...
```

---

## 2. Maximum XOR (LC 421)

For each number, walk trie preferring **opposite bit** at each level.

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
function findMaximumXOR(nums) {
  const root = { zero: null, one: null };
  const BITS = 31;

  for (const num of nums) {
    let node = root;
    for (let i = BITS; i >= 0; i--) {
      const bit = (num >> i) & 1;
      if (bit === 0) {
        if (!node.zero) node.zero = { zero: null, one: null };
        node = node.zero;
      } else {
        if (!node.one) node.one = { zero: null, one: null };
        node = node.one;
      }
    }
  }

  let maxXor = 0;
  for (const num of nums) {
    let node = root;
    let cur = 0;
    for (let i = BITS; i >= 0; i--) {
      const bit = (num >> i) & 1;
      const want = bit ^ 1;
      if (want === 0 && node.zero) { cur |= 1 << i; node = node.zero; }
      else if (want === 1 && node.one) { cur |= 1 << i; node = node.one; }
      else if (node.zero) node = node.zero;
      else node = node.one;
    }
    maxXor = Math.max(maxXor, cur);
  }
  return maxXor;
}
```

| Time | Space |
|------|-------|
| O(n × 32) | O(n × 32) |

---

## 3. IP Routing Table

Longest prefix match — store CIDR routes in binary trie:

```js
function ipToBinary(ip) {
  return ip.split(".").map((o) => (+o).toString(2).padStart(8, "0")).join("");
}

class IPRoutingTable {
  constructor() {
    this.root = { zero: null, one: null, route: null };
  }

  addRoute(cidr, handler) {
    const [ip, prefixLen] = cidr.split("/");
    const bits = ipToBinary(ip).slice(0, +prefixLen);
    let node = this.root;
    for (const b of bits) {
      if (b === "0") {
        if (!node.zero) node.zero = { zero: null, one: null, route: null };
        node = node.zero;
      } else {
        if (!node.one) node.one = { zero: null, one: null, route: null };
        node = node.one;
      }
    }
    node.route = handler;
  }

  lookup(ip) {
    const bits = ipToBinary(ip);
    let node = this.root;
    let best = null;
    for (let i = 0; i < bits.length && node; i++) {
      if (node.route) best = node.route;
      node = bits[i] === "0" ? node.zero : node.one;
    }
    if (node?.route) best = node.route;
    return best;
  }
}

const rt = new IPRoutingTable();
rt.addRoute("192.168.0.0/16", "LAN");
rt.addRoute("192.168.1.0/24", "Subnet-A");
rt.lookup("192.168.1.42"); // "Subnet-A" (longest prefix)
```

---

## 4. Range Bitwise AND (LC 1707)

Maximum bitwise AND of `num[i] AND x` for any x in `[val1, val2]`.

**Binary trie** storing nums; for query range, traverse preferring bits that keep x in range.

```js
function maximumAND(nums, queries) {
  nums.sort((a, b) => a - b);
  const answer = [];
  for (const [val, mini, maxi] of queries) {
    let best = 0;
    // Build trie of nums <= val (offline sort queries trick)
    // Simplified: brute best x in range for interview clarity
    for (let x = maxi; x >= mini; x--) {
      let cur = 0;
      for (const n of nums) {
        if (n > val) break;
        cur = Math.max(cur, n & x);
      }
      best = Math.max(best, cur);
    }
    answer.push(best);
  }
  return answer;
}
```

**Production approach:** Sort queries offline; binary trie with greedy bit selection respecting `[mini, maxi]`.

---

## 5. Max Genetic Score (LC 1938)

Tree of parents + gene values. Queries: max `(score & mask)` in subtree.

**Binary trie per subtree** (merge small-to-large) or store all values in subtree trie.

```js
// Conceptual: for each node, binary trie of all descendant scores
function getMaxGeneticScore(parents, scores, queries) {
  const n = scores.length;
  const children = Array.from({ length: n }, () => []);
  for (let i = 1; i < n; i++) children[parents[i]].push(i);

  const trie = buildSubtreeTrie(0, children, scores);
  // Answer queries with mask by greedy opposite-bit walk
  return queries.map(([node, mask]) => queryMaxAND(trie[node], mask));
}
```

---

## 6. Network Prefix Match (RN)

Mobile app checks if device IP matches corporate VPN CIDR before allowing sync:

```js
function isInNetwork(deviceIp, cidrList) {
  const table = new IPRoutingTable();
  cidrList.forEach((c) => table.addRoute(c, true));
  return table.lookup(deviceIp) !== null;
}

// RN usage with expo-network or native module
async function shouldUseLocalSync() {
  const ip = await getDeviceIp(); // platform-specific
  return isInNetwork(ip, ["10.0.0.0/8", "172.16.0.0/12"]);
}
```

---

## 7. Interview Q&A

### Q1: Why binary trie for XOR?

**A:** Greedy bit choice at each level — opposite bit maximizes XOR contribution at that position.

### Q2: IP longest prefix match?

**A:** Track best route at every trie level during walk; deeper match overrides.

### Q3: 32 vs 31 bits?

**A:** JS numbers safe to 2^53; for 32-bit ints use bits 31→0. Handle sign if needed.

### Q4: Binary trie vs hash for XOR?

**A:** Hash can't greedily maximize per bit; trie O(32) per query.

### Q5: Delete from binary trie?

**A:** Decrement count on path; prune nodes with count 0.

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| Binary trie insert | [§1](#1-binary-trie-structure) |
| Max XOR | [§2](#2-maximum-xor-lc-421) |
| IP routing | [§3](#3-ip-routing-table) |
| RN network check | [§6](#6-network-prefix-match-rn) |

---

## Day 48 Cheat Sheet

```
Binary trie:  32 levels, bit 0/1 children
Max XOR:      prefer opposite bit at each level
IP routing:   CIDR prefix insert, longest match lookup
LC 1707:      trie + range constraint on x
LC 1938:      subtree trie + mask query
```

---

*End of Day 48 concepts*
