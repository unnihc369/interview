# Day 45 — Machine Coding Revision

**React:** Word game prefix/suffix filter  
**React Native:** Predictive text keyboard

---

## Table of Contents

### React (Web)

1. [Problem — Word Game Helper](#1-problem--word-game-helper)
2. [Trie + Prefix/Suffix Filter](#2-trie--prefixsuffix-filter)
3. [React Word Game UI](#3-react-word-game-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Predictive Text Input](#5-predictive-text-input)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Word Game Helper

**Task:** Word-guess helper. User sets **prefix** (known start), **suffix** (known end), **length**, and **excluded letters**. Filter valid words from dictionary trie.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Prefix filter | startsWith |
| Suffix filter | endsWith via reversed trie |
| Length filter | exact length |
| Excluded | must not contain gray letters |

---

## 2. Trie + Prefix/Suffix Filter

```js
class WordGameTrie {
  constructor() {
    this.prefixRoot = { children: new Map(), isEnd: false, word: null };
    this.suffixRoot = { children: new Map(), isEnd: false };
  }

  insert(word) {
    let p = this.prefixRoot;
    for (const ch of word) {
      if (!p.children.has(ch)) p.children.set(ch, { children: new Map(), isEnd: false, word: null });
      p = p.children.get(ch);
    }
    p.isEnd = true;
    p.word = word;

    let s = this.suffixRoot;
    for (let i = word.length - 1; i >= 0; i--) {
      const ch = word[i];
      if (!s.children.has(ch)) s.children.set(ch, { children: new Map(), isEnd: false });
      s = s.children.get(ch);
    }
    s.isEnd = true;
  }

  collectPrefix(prefix) {
    let node = this.prefixRoot;
    for (const ch of prefix) {
      if (!node.children.has(ch)) return [];
      node = node.children.get(ch);
    }
    const out = [];
    const dfs = (n) => {
      if (n.word) out.push(n.word);
      for (const child of n.children.values()) dfs(child);
    };
    dfs(node);
    return out;
  }

  matchesSuffix(word, suffix) {
    if (!suffix) return true;
    let node = this.suffixRoot;
    for (let i = suffix.length - 1; i >= 0; i--) {
      if (!node.children.has(suffix[i])) return false;
      node = node.children.get(suffix[i]);
    }
    return node.isEnd;
  }

  filter({ prefix = "", suffix = "", length = 0, exclude = "" }) {
    const candidates = prefix ? this.collectPrefix(prefix) : this.allWords();
    const exSet = new Set(exclude.toLowerCase());
    return candidates.filter((w) => {
      if (length && w.length !== length) return false;
      if (suffix && !this.matchesSuffix(w, suffix)) return false;
      for (const ch of exSet) if (w.includes(ch)) return false;
      return true;
    });
  }

  allWords() {
    const out = [];
    const dfs = (n) => {
      if (n.word) out.push(n.word);
      for (const c of n.children.values()) dfs(c);
    };
    dfs(this.prefixRoot);
    return out;
  }
}

const WORDS = ["apple","apply","ample","angle","ankle","apples","app","banana","candle","handle"];
const gameTrie = new WordGameTrie();
WORDS.forEach((w) => gameTrie.insert(w));
```

---

## 3. React Word Game UI

```tsx
import { useState, useMemo } from "react";

export default function WordGameHelper() {
  const [prefix, setPrefix] = useState("ap");
  const [suffix, setSuffix] = useState("");
  const [length, setLength] = useState(5);
  const [exclude, setExclude] = useState("y");

  const matches = useMemo(
    () => gameTrie.filter({ prefix, suffix, length, exclude }),
    [prefix, suffix, length, exclude]
  );

  return (
    <div style={{ maxWidth: 480, margin: 24, fontFamily: "system-ui" }}>
      <h2>Word Game Filter</h2>
      <div style={{ display: "grid", gap: 10 }}>
        <label>Prefix <input value={prefix} onChange={(e) => setPrefix(e.target.value)} /></label>
        <label>Suffix <input value={suffix} onChange={(e) => setSuffix(e.target.value)} /></label>
        <label>Length <input type="number" value={length} onChange={(e) => setLength(+e.target.value)} /></label>
        <label>Exclude <input value={exclude} onChange={(e) => setExclude(e.target.value)} /></label>
      </div>
      <p style={{ marginTop: 16 }}>{matches.length} matches</p>
      <div style={{ display: "flex", flexWrap: "wrap", gap: 8 }}>
        {matches.map((w) => (
          <span key={w} style={{ padding: "6px 12px", background: "#e3f2fd", borderRadius: 6 }}>{w}</span>
        ))}
      </div>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Suffix without reversed trie? | endsWith check O(m) per word |
| Reversed trie benefit | Early prune suffix mismatch |
| Wordle optimization | Filter by length first, then trie prefix |

---

# Part B — React Native

## 5. Predictive Text Input

**Task:** Message input with **word suggestions** above keyboard. Trie built from recent messages + dictionary. Lazy-delete word on user dismissing suggestion.

```tsx
import { useState, useMemo, useRef } from "react";
import {
  View, TextInput, Text, Pressable, FlatList, StyleSheet,
} from "react-native";

class PredictiveTrie {
  root = { children: new Map(), isEnd: false, freq: 0 };

  insert(word, freq = 1) {
    let node = this.root;
    for (const ch of word.toLowerCase()) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), isEnd: false, freq: 0 });
      node = node.children.get(ch);
      node.freq += freq;
    }
    node.isEnd = true;
  }

  deleteLazy(word) {
    let node = this.root;
    for (const ch of word.toLowerCase()) {
      if (!node.children.has(ch)) return;
      node = node.children.get(ch);
      node.freq = Math.max(0, node.freq - 1);
    }
    node.isEnd = false;
  }

  suggest(prefix, limit = 3) {
    let node = this.root;
    for (const ch of prefix.toLowerCase()) {
      if (!node.children.has(ch)) return [];
      node = node.children.get(ch);
    }
    const out = [];
    const dfs = (n, path) => {
      if (out.length >= limit) return;
      if (n.isEnd) out.push({ word: path, freq: n.freq });
      for (const [ch, child] of n.children) dfs(child, path + ch);
    };
    dfs(node, prefix.toLowerCase());
    return out.sort((a, b) => b.freq - a.freq).slice(0, limit);
  }
}

const predictiveTrie = new PredictiveTrie();
["hello", "help", "hey", "world", "work", "want"].forEach((w) => predictiveTrie.insert(w, 5));
predictiveTrie.insert("hello", 10); // boost frequency

export default function PredictiveTextInput() {
  const [text, setText] = useState("");
  const currentWord = text.split(/\s/).pop() ?? "";

  const suggestions = useMemo(() => {
    if (currentWord.length < 2) return [];
    return predictiveTrie.suggest(currentWord, 3);
  }, [currentWord]);

  const applySuggestion = (word: string) => {
    const parts = text.split(/\s/);
    parts[parts.length - 1] = word;
    setText(parts.join(" ") + " ");
  };

  const dismiss = (word: string) => {
    predictiveTrie.deleteLazy(word);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Predictive Text</Text>
      {suggestions.length > 0 && (
        <View style={styles.chips}>
          {suggestions.map(({ word }) => (
            <Pressable key={word} style={styles.chip} onPress={() => applySuggestion(word)}
              onLongPress={() => dismiss(word)}>
              <Text>{word}</Text>
            </Pressable>
          ))}
        </View>
      )}
      <TextInput
        value={text}
        onChangeText={setText}
        placeholder="Type a message…"
        style={styles.input}
        multiline
      />
      <Text style={styles.hint}>Long-press chip to dismiss (lazy delete)</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, paddingTop: 60 },
  title: { fontSize: 18, fontWeight: "700", marginBottom: 12 },
  chips: { flexDirection: "row", gap: 8, marginBottom: 8 },
  chip: { paddingHorizontal: 14, paddingVertical: 8, backgroundColor: "#e8eaf6", borderRadius: 20 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 10, padding: 14, minHeight: 80, fontSize: 16 },
  hint: { fontSize: 11, color: "#888", marginTop: 8 },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Lazy delete in keyboard | User dismisses bad suggestion — mark isEnd false |
| Frequency ranking | Store freq at terminal; sort suggestions |
| Native keyboard integration | Custom InputAccessoryView (iOS) for suggestion bar |
| Learn from typing | insert on send with boosted freq |

---

## Quick Revision — Day 45

```
Web:  Prefix + reversed suffix trie word filter
RN:   Predictive chips + lazy delete + freq rank
LC:   1268 suggestions, 1032 stream, 676 magic dict
```

---

*End of Day 45 machine coding*
