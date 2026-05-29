# Day 47 — Machine Coding Revision

**React:** Text search highlighter  
**React Native:** Syntax highlighter prefix match

---

## Table of Contents

### React (Web)

1. [Problem — In-Document Search Highlight](#1-problem--in-document-search-highlight)
2. [Suffix Trie Search](#2-suffix-trie-search)
3. [React Highlighter Component](#3-react-highlighter-component)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Syntax Highlighter](#5-syntax-highlighter)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — In-Document Search Highlight

**Task:** Display a paragraph with **live search**. Highlight all occurrences of query (case-insensitive). Show match count.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Live highlight | on input change |
| Case insensitive | normalize |
| Match count | display badge |
| Scroll to first | optional |

---

## 2. Suffix Trie Search

```js
function findAllIndices(text, query) {
  if (!query) return [];
  const lower = text.toLowerCase();
  const q = query.toLowerCase();
  const indices = [];
  let pos = 0;
  while (true) {
    const idx = lower.indexOf(q, pos);
    if (idx === -1) break;
    indices.push(idx);
    pos = idx + 1; // allow overlapping
  }
  return indices;
}

function buildHighlightedParts(text, query) {
  if (!query.trim()) return [{ text, highlight: false }];
  const indices = findAllIndices(text, query);
  if (!indices.length) return [{ text, highlight: false }];

  const qLen = query.length;
  const highlightSet = new Set(indices);

  const parts = [];
  for (let i = 0; i < text.length; ) {
    if (highlightSet.has(i)) {
      parts.push({ text: text.slice(i, i + qLen), highlight: true });
      i += qLen;
    } else {
      let j = i + 1;
      while (j < text.length && !highlightSet.has(j)) j++;
      parts.push({ text: text.slice(i, j), highlight: false });
      i = j;
    }
  }
  return parts;
}

const SAMPLE =
  "The trie data structure supports efficient prefix search. " +
  "A suffix trie stores all suffixes for substring search.";
```

---

## 3. React Highlighter Component

```tsx
import { useState, useMemo, useRef, useEffect } from "react";

export default function TextSearchHighlighter() {
  const [query, setQuery] = useState("prefix");
  const firstMatchRef = useRef<HTMLSpanElement>(null);

  const parts = useMemo(() => buildHighlightedParts(SAMPLE, query), [query]);
  const matchCount = useMemo(() => findAllIndices(SAMPLE, query).length, [query]);

  useEffect(() => {
    firstMatchRef.current?.scrollIntoView({ behavior: "smooth", block: "nearest" });
  }, [query]);

  let firstMarked = false;

  return (
    <div style={{ maxWidth: 560, margin: 24, fontFamily: "Georgia, serif" }}>
      <h2>Text Search Highlighter</h2>
      <div style={{ display: "flex", gap: 12, alignItems: "center", marginBottom: 16 }}>
        <input
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search…"
          style={{ flex: 1, padding: 10, fontSize: 16, borderRadius: 8, border: "1px solid #ccc" }}
        />
        <span style={{
          padding: "4px 10px", background: matchCount ? "#e3f2fd" : "#f5f5f5",
          borderRadius: 12, fontSize: 13,
        }}>
          {matchCount} match{matchCount !== 1 ? "es" : ""}
        </span>
      </div>
      <p style={{ lineHeight: 1.7, fontSize: 17 }}>
        {parts.map((p, i) => {
          const isFirst = p.highlight && !firstMarked;
          if (isFirst) firstMarked = true;
          return p.highlight ? (
            <mark
              key={i}
              ref={isFirst ? firstMatchRef : undefined}
              style={{ background: "#fff59d", padding: "0 2px" }}
            >
              {p.text}
            </mark>
          ) : (
            <span key={i}>{p.text}</span>
          );
        })}
      </p>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| indexOf vs suffix trie | indexOf fine for single doc; trie for many queries |
| Overlapping matches | Advance pos by 1 not qLen |
| Performance large doc | Web Worker + suffix array |
| Regex special chars | Escape query before RegExp |

---

# Part B — React Native

## 5. Syntax Highlighter

**Task:** Code block with **keyword highlighting** via prefix trie longest match.

```tsx
import { useMemo } from "react";
import { View, Text, ScrollView, StyleSheet } from "react-native";

class KeywordTrie {
  root = { children: new Map(), isEnd: false, word: "" };
  insert(w: string) {
    let n = this.root;
    for (const ch of w) {
      if (!n.children.has(ch)) n.children.set(ch, { children: new Map(), isEnd: false, word: "" });
      n = n.children.get(ch)!;
    }
    n.isEnd = true;
    n.word = w;
  }
  longestMatch(code: string, start: number) {
    let n = this.root, matched = "", i = start;
    while (i < code.length && n.children.has(code[i])) {
      n = n.children.get(code[i])!;
      i++;
      if (n.isEnd) matched = n.word;
    }
    return matched;
  }
}

const kwTrie = new KeywordTrie();
["const","let","function","return","if","else","async","await","import","export"].forEach((k) => kwTrie.insert(k));

function tokenize(code: string) {
  const tokens: { type: string; value: string }[] = [];
  let i = 0;
  while (i < code.length) {
    const ch = code[i];
    if (/\s/.test(ch)) {
      let j = i + 1;
      while (j < code.length && /\s/.test(code[j])) j++;
      tokens.push({ type: "whitespace", value: code.slice(i, j) });
      i = j;
      continue;
    }
    if (/[a-zA-Z_]/.test(ch)) {
      let j = i + 1;
      while (j < code.length && /[a-zA-Z0-9_]/.test(code[j])) j++;
      const word = code.slice(i, j);
      const kw = kwTrie.longestMatch(code, i);
      tokens.push({ type: kw && word.startsWith(kw) ? "keyword" : "identifier", value: word });
      i = j;
      continue;
    }
    tokens.push({ type: "punct", value: ch });
    i++;
  }
  return tokens;
}

const CODE = `async function fetchData() {
  const response = await fetch(url);
  if (!response.ok) return null;
  return response.json();
}`;

const COLORS: Record<string, string> = {
  keyword: "#c792ea",
  identifier: "#eeffff",
  whitespace: "#eeffff",
  punct: "#89ddff",
};

export default function SyntaxHighlighter() {
  const tokens = useMemo(() => tokenize(CODE), []);

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Syntax Highlighter (Trie)</Text>
      <View style={styles.codeBlock}>
        <Text style={styles.code}>
          {tokens.map((t, i) => (
            <Text key={i} style={{ color: COLORS[t.type] }}>{t.value}</Text>
          ))}
        </Text>
      </View>
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, paddingTop: 56, backgroundColor: "#263238" },
  title: { color: "#fff", fontSize: 18, fontWeight: "700", marginBottom: 12 },
  codeBlock: { padding: 16, borderRadius: 8, backgroundColor: "#1e272c" },
  code: { fontFamily: "monospace", fontSize: 14, lineHeight: 22 },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Nested Text styling | RN requires nested `<Text>` for multi-color |
| vs CodeMirror/Monaco | Native Text + trie for lightweight; WebView for full IDE |
| Longest prefix match | Trie beats Set lookup for partial keywords |
| Performance | Tokenize once; trie built at module load |

---

## Quick Revision — Day 47

```
Web:  Search highlight + match count + scroll to first
RN:   Keyword trie tokenize + nested Text colors
LC:   745 WordFilter, 1062/1044 duplicate substring
```

---

*End of Day 47 machine coding*
