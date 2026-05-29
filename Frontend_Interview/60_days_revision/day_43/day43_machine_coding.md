# Day 43 — Machine Coding Revision

**React:** Autocomplete input with basic trie  
**React Native:** Search bar suggestions

---

## Table of Contents

### React (Web)

1. [Problem — Trie Autocomplete](#1-problem--trie-autocomplete)
2. [Trie Implementation](#2-trie-implementation)
3. [React Autocomplete UI](#3-react-autocomplete-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Search Bar Suggestions](#5-search-bar-suggestions)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Trie Autocomplete

**Task:** Build a search input that shows **prefix-based suggestions** from a static corpus using a Trie.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Prefix match | case-insensitive |
| Max 5 suggestions | DFS from prefix node |
| Debounce | 200ms on input |
| Keyboard | Arrow up/down, Enter, Escape |
| a11y | combobox + listbox roles |

---

## 2. Trie Implementation

```js
class TrieNode {
  constructor() {
    this.children = new Map();
    this.isEnd = false;
  }
}

class AutocompleteTrie {
  constructor() {
    this.root = new TrieNode();
  }

  insert(word) {
    const w = word.toLowerCase();
    let node = this.root;
    for (const ch of w) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNode());
      node = node.children.get(ch);
    }
    node.isEnd = true;
  }

  suggest(prefix, limit = 5) {
    const p = prefix.toLowerCase();
    let node = this.root;
    for (const ch of p) {
      if (!node.children.has(ch)) return [];
      node = node.children.get(ch);
    }

    const out = [];
    const dfs = (n, path) => {
      if (out.length >= limit) return;
      if (n.isEnd) out.push(path);
      for (const [ch, child] of [...n.children.entries()].sort()) {
        dfs(child, path + ch);
      }
    };
    dfs(node, p);
    return out;
  }
}

const CORPUS = [
  "javascript", "java", "json", "jest", "jsx", "jquery",
  "react", "redux", "router", "typescript", "tailwind",
];

const trie = new AutocompleteTrie();
CORPUS.forEach((w) => trie.insert(w));
```

---

## 3. React Autocomplete UI

```tsx
import { useState, useMemo, useEffect, useCallback } from "react";

function useDebounced<T>(value: T, ms: number): T {
  const [v, setV] = useState(value);
  useEffect(() => {
    const t = setTimeout(() => setV(value), ms);
    return () => clearTimeout(t);
  }, [value, ms]);
  return v;
}

export default function TrieAutocomplete() {
  const [query, setQuery] = useState("");
  const [active, setActive] = useState(0);
  const [open, setOpen] = useState(false);
  const debounced = useDebounced(query, 200);

  const suggestions = useMemo(() => {
    if (!debounced.trim()) return [];
    return trie.suggest(debounced, 5);
  }, [debounced]);

  useEffect(() => setActive(0), [debounced]);

  const select = useCallback((term: string) => {
    setQuery(term);
    setOpen(false);
  }, []);

  const onKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === "Escape") { setOpen(false); return; }
    if (!suggestions.length) return;
    if (e.key === "ArrowDown") {
      e.preventDefault();
      setActive((i) => Math.min(i + 1, suggestions.length - 1));
    } else if (e.key === "ArrowUp") {
      e.preventDefault();
      setActive((i) => Math.max(i - 1, 0));
    } else if (e.key === "Enter") {
      e.preventDefault();
      select(suggestions[active]);
    }
  };

  return (
    <div style={{ maxWidth: 400, margin: 24, position: "relative" }}>
      <h2>Trie Autocomplete</h2>
      <input
        value={query}
        onChange={(e) => { setQuery(e.target.value); setOpen(true); }}
        onFocus={() => setOpen(true)}
        onBlur={() => setTimeout(() => setOpen(false), 150)}
        onKeyDown={onKeyDown}
        placeholder="Search tech terms…"
        role="combobox"
        aria-expanded={open && suggestions.length > 0}
        aria-autocomplete="list"
        style={{ width: "100%", padding: 12, fontSize: 16, borderRadius: 8, border: "1px solid #ccc" }}
      />
      {open && suggestions.length > 0 && (
        <ul
          role="listbox"
          style={{
            position: "absolute", width: "100%", margin: 4, 0, 0, padding: 0,
            listStyle: "none", border: "1px solid #ddd", borderRadius: 8,
            background: "#fff", boxShadow: "0 4px 12px rgba(0,0,0,0.1)", zIndex: 10,
          }}
        >
          {suggestions.map((s, i) => (
            <li
              key={s}
              role="option"
              aria-selected={i === active}
              onMouseDown={() => select(s)}
              style={{
                padding: "10px 14px",
                cursor: "pointer",
                background: i === active ? "#e8f4fd" : "#fff",
              }}
            >
              <Highlight prefix={debounced.toLowerCase()} term={s} />
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

function Highlight({ prefix, term }: { prefix: string; term: string }) {
  if (!prefix || !term.startsWith(prefix)) return <>{term}</>;
  return (
    <>
      <strong>{term.slice(0, prefix.length)}</strong>
      {term.slice(prefix.length)}
    </>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Why trie over filter+sort? | O(m) prefix vs O(n) scan entire corpus |
| Debounce value | 150–300ms API; 0–200ms local trie |
| onMouseDown vs onClick | mousedown fires before input blur |
| Scale to API | Local trie for hot terms; remote for rest |
| Memory | ~1M terms ≈ tens of MB with Map nodes — acceptable |

---

# Part B — React Native

## 5. Search Bar Suggestions

**Task:** RN search screen with trie-backed suggestions, FlatList dropdown, and keyboard-safe layout.

```tsx
import { useState, useMemo, useEffect } from "react";
import {
  View, TextInput, FlatList, Text, Pressable,
  StyleSheet, KeyboardAvoidingView, Platform,
} from "react-native";

// Reuse AutocompleteTrie from above
const rnTrie = new AutocompleteTrie();
["iphone", "ipad", "imac", "airpods", "apple watch", "macbook"].forEach((w) =>
  rnTrie.insert(w)
);

export default function SearchBarSuggestions() {
  const [query, setQuery] = useState("");
  const suggestions = useMemo(() => {
    if (query.length < 1) return [];
    return rnTrie.suggest(query, 6);
  }, [query]);

  return (
    <KeyboardAvoidingView
      style={styles.container}
      behavior={Platform.OS === "ios" ? "padding" : undefined}
    >
      <Text style={styles.title}>Product Search</Text>
      <TextInput
        value={query}
        onChangeText={setQuery}
        placeholder="Search products…"
        style={styles.input}
        autoCorrect={false}
        autoCapitalize="none"
        returnKeyType="search"
      />
      {suggestions.length > 0 && (
        <FlatList
          data={suggestions}
          keyExtractor={(item) => item}
          keyboardShouldPersistTaps="handled"
          style={styles.list}
          renderItem={({ item }) => (
            <Pressable
              style={styles.row}
              onPress={() => setQuery(item)}
            >
              <Text style={styles.rowText}>{item}</Text>
            </Pressable>
          )}
        />
      )}
      {query.length > 0 && suggestions.length === 0 && (
        <Text style={styles.empty}>No matches for "{query}"</Text>
      )}
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, paddingTop: 60 },
  title: { fontSize: 20, fontWeight: "700", marginBottom: 12 },
  input: {
    borderWidth: 1, borderColor: "#ccc", borderRadius: 10,
    padding: 14, fontSize: 16, backgroundColor: "#fafafa",
  },
  list: { marginTop: 8, maxHeight: 240, borderRadius: 10, borderWidth: 1, borderColor: "#eee" },
  row: { padding: 14, borderBottomWidth: StyleSheet.hairlineWidth, borderBottomColor: "#eee" },
  rowText: { fontSize: 16 },
  empty: { marginTop: 12, color: "#888", fontSize: 14 },
});
```

### Performance notes

- Build trie once at module load or in `useMemo([])`.
- `keyboardShouldPersistTaps="handled"` so taps register while keyboard open.
- For large corpora, persist trie JSON to AsyncStorage and hydrate on launch.

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| FlatList vs ScrollView | FlatList virtualizes long suggestion lists |
| KeyboardAvoidingView | iOS padding; Android often `android:windowSoftInputMode` |
| Persist trie offline | Serialize to AsyncStorage; rebuild on first launch |
| Native search bar | `headerSearchBarOptions` (iOS) vs custom TextInput |

---

## Quick Revision — Day 43

```
Web:  Trie insert + DFS suggest + debounced combobox
RN:   TextInput + FlatList suggestions + keyboardShouldPersistTaps
LC:   208 Trie, 211 Wildcard, 212 Board + Trie
```

---

*End of Day 43 machine coding*
