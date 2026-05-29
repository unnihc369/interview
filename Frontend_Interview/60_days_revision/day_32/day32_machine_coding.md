# Day 32 — Machine Coding Revision

**React:** Save/load tree from localStorage · import/export JSON  
**React Native:** Tree diff sync · AsyncStorage · merge strategies

---

## Table of Contents

1. [localStorage Tree Persistence — React](#1-localstorage-tree-persistence--react)
2. [usePersistedTree Hook](#2-usepersistedtree-hook)
3. [Import/Export UI](#3-importexport-ui)
4. [Tree Diff — React Native](#4-tree-diff--react-native)
5. [Merge Strategy](#5-merge-strategy)
6. [Interview Checklist](#6-interview-checklist)

---

## 1. localStorage Tree Persistence — React

```ts
const STORAGE_KEY = "folder-tree-v1";

type TreeJSON = { id: string; name: string; type: string; children?: TreeJSON[] };

function saveTree(tree: FileNode) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tree));
}

function loadTree(): FileNode | null {
  const raw = localStorage.getItem(STORAGE_KEY);
  return raw ? JSON.parse(raw) : null;
}
```

---

## 2. usePersistedTree Hook

```tsx
function usePersistedTree(initial: FileNode, key = STORAGE_KEY) {
  const [tree, setTree] = useState<FileNode>(() => {
    const saved = localStorage.getItem(key);
    return saved ? JSON.parse(saved) : initial;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(tree));
  }, [tree, key]);

  const reset = () => {
    localStorage.removeItem(key);
    setTree(initial);
  };

  return { tree, setTree, reset };
}
```

---

## 3. Import/Export UI

```tsx
function TreePersistencePanel({ tree, setTree }: { tree: FileNode; setTree: (t: FileNode) => void }) {
  const exportJSON = () => {
    const blob = new Blob([JSON.stringify(tree, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "tree.json";
    a.click();
  };

  const importJSON = (file: File) => {
    const reader = new FileReader();
    reader.onload = () => setTree(JSON.parse(reader.result as string));
    reader.readAsText(file);
  };

  return (
    <div style={{ display: "flex", gap: 8 }}>
      <button onClick={exportJSON}>Export JSON</button>
      <input type="file" accept=".json" onChange={(e) => e.target.files?.[0] && importJSON(e.target.files[0])} />
    </div>
  );
}
```

---

## 4. Tree Diff — React Native

```ts
import AsyncStorage from "@react-native-async-storage/async-storage";

type DiffResult = { added: string[]; removed: string[]; changed: string[] };

function diffTrees(oldIds: Set<string>, newIds: Set<string>): DiffResult {
  return {
    added: [...newIds].filter((id) => !oldIds.has(id)),
    removed: [...oldIds].filter((id) => !newIds.has(id)),
    changed: [], // extend with deep compare on shared ids
  };
}

function collectIds(node: FileNode): Set<string> {
  const ids = new Set([node.id]);
  node.children?.forEach((c) => collectIds(c).forEach((id) => ids.add(id)));
  return ids;
}
```

---

## 5. Merge Strategy

```ts
function mergeTrees(local: FileNode, remote: FileNode): FileNode {
  // Last-write-wins on name; union children by id
  const childMap = new Map<string, FileNode>();
  local.children?.forEach((c) => childMap.set(c.id, c));
  remote.children?.forEach((c) => {
    childMap.set(c.id, childMap.has(c.id) ? mergeTrees(childMap.get(c.id)!, c) : c);
  });
  return {
    ...remote,
    name: remote.name || local.name,
    children: [...childMap.values()],
  };
}

async function syncTree(local: FileNode, fetchRemote: () => Promise<FileNode>) {
  const remote = await fetchRemote();
  const merged = mergeTrees(local, remote);
  await AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(merged));
  return merged;
}
```

---

## 6. Interview Checklist

- [ ] Serialize to JSON for storage
- [ ] useEffect debounce or save on change
- [ ] Export/import file download
- [ ] RN AsyncStorage equivalent
- [ ] Mention diff/merge for offline-first apps

---

*End of Day 32 machine coding*
