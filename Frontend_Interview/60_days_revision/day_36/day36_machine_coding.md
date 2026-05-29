# Day 36 — Machine Coding Revision

**React:** Live BST visualizer · insert/search highlight  
**React Native:** Dictionary BST index · autocomplete

---

## Table of Contents

1. [BST Visualizer State — React](#1-bst-visualizer-state--react)
2. [Animated Insert & Search](#2-animated-insert--search)
3. [Canvas/SVG Tree Layout](#3-canvassvg-tree-layout)
4. [Dictionary Index — RN](#4-dictionary-index--rn)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. BST Visualizer State — React

```tsx
type BSTState = { root: TreeNode | null; lastVisited: number[] };

function bstReducer(state: BSTState, action: Action): BSTState {
  switch (action.type) {
    case "INSERT":
      return { root: insertIntoBST(state.root, action.val), lastVisited: action.path };
    case "SEARCH":
      return { ...state, lastVisited: findPath(state.root, action.val) };
    case "RESET":
      return { root: null, lastVisited: [] };
    default:
      return state;
  }
}

function findPath(root: TreeNode | null, val: number): number[] {
  const path: number[] = [];
  let cur = root;
  while (cur) {
    path.push(cur.val);
    if (val === cur.val) return path;
    cur = val < cur.val ? cur.left : cur.right;
  }
  return path;
}
```

---

## 2. Animated Insert & Search

```tsx
function BSTVisualizer() {
  const [state, dispatch] = useReducer(bstReducer, { root: null, lastVisited: [] });
  const [input, setInput] = useState("");

  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={() => dispatch({ type: "INSERT", val: +input, path: [] })}>Insert</button>
      <button onClick={() => dispatch({ type: "SEARCH", val: +input })}>Search</button>
      <TreeCanvas root={state.root} highlight={new Set(state.lastVisited)} />
    </div>
  );
}

function TreeNodeView({ node, highlight }: { node: TreeNode; highlight: Set<number> }) {
  return (
    <div style={{ textAlign: "center" }}>
      <div
        style={{
          width: 36, height: 36, borderRadius: "50%",
          background: highlight.has(node.val) ? "#fbbf24" : "#e5e7eb",
          display: "inline-flex", alignItems: "center", justifyContent: "center",
        }}
      >
        {node.val}
      </div>
      {(node.left || node.right) && (
        <div style={{ display: "flex", justifyContent: "center", gap: 32, marginTop: 16 }}>
          {node.left && <TreeNodeView node={node.left} highlight={highlight} />}
          {node.right && <TreeNodeView node={node.right} highlight={highlight} />}
        </div>
      )}
    </div>
  );
}
```

---

## 3. Canvas/SVG Tree Layout

For interviews, recursive flex layout is enough. Mention **Reingold-Tilford** for production tree layout.

---

## 4. Dictionary Index — RN

```ts
class WordNode {
  word: string;
  left: WordNode | null = null;
  right: WordNode | null = null;
  constructor(word: string) { this.word = word; }
}

function insertWord(root: WordNode | null, word: string): WordNode {
  if (!root) return new WordNode(word);
  if (word < root.word) root.left = insertWord(root.left, word);
  else if (word > root.word) root.right = insertWord(root.right, word);
  return root;
}

function prefixSearch(root: WordNode | null, prefix: string): string[] {
  const results: string[] = [];
  function dfs(node: WordNode | null) {
    if (!node || results.length >= 10) return;
    if (node.word.startsWith(prefix)) results.push(node.word);
    if (node.word < prefix) dfs(node.right);
    else { dfs(node.left); dfs(node.right); }
  }
  dfs(root);
  return results;
}
```

```tsx
function DictionarySearch() {
  const [root, setRoot] = useState<WordNode | null>(null);
  const [query, setQuery] = useState("");
  const suggestions = useMemo(() => prefixSearch(root, query), [root, query]);

  return (
    <View>
      <TextInput value={query} onChangeText={setQuery} placeholder="Search word..." />
      <FlatList data={suggestions} keyExtractor={(w) => w} renderItem={({ item }) => <Text>{item}</Text>} />
    </View>
  );
}
```

---

## 5. Interview Checklist

- [ ] Insert + search with path highlight
- [ ] BST property maintained on insert
- [ ] Autocomplete from BST/trie
- [ ] Explain O(log n) vs O(n) skewed case

---

*End of Day 36 machine coding*
