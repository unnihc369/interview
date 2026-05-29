# Day 33 — Machine Coding Revision

**React:** Family tree LCA · highlight shared ancestor  
**React Native:** Message thread LCA · jump to fork point

---

## Table of Contents

1. [Family Tree Data Model](#1-family-tree-data-model)
2. [Find LCA in UI Tree](#2-find-lca-in-ui-tree)
3. [Highlight Path to LCA — React](#3-highlight-path-to-lca--react)
4. [Message Thread LCA — RN](#4-message-thread-lca--rn)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Family Tree Data Model

```ts
type Person = {
  id: string;
  name: string;
  parentId: string | null;
  children: Person[];
};

function buildTree(flat: Omit<Person, "children">[]): Person {
  const map = new Map(flat.map((p) => [p.id, { ...p, children: [] as Person[] }]));
  let root!: Person;
  for (const p of map.values()) {
    if (p.parentId) map.get(p.parentId)!.children.push(p);
    else root = p;
  }
  return root;
}
```

---

## 2. Find LCA in UI Tree

```ts
function findLCA(root: Person, idA: string, idB: string): Person | null {
  if (!root) return null;
  if (root.id === idA || root.id === idB) return root;
  const found = root.children
    .map((c) => findLCA(c, idA, idB))
    .filter(Boolean) as Person[];
  if (found.length === 2) return root;
  return found[0] ?? null;
}

function pathToNode(root: Person, targetId: string, path: Person[] = []): Person[] | null {
  path.push(root);
  if (root.id === targetId) return path;
  for (const child of root.children) {
    const result = pathToNode(child, targetId, [...path]);
    if (result) return result;
  }
  return null;
}
```

---

## 3. Highlight Path to LCA — React

```tsx
function FamilyTreeViewer({ root }: { root: Person }) {
  const [selected, setSelected] = useState<[string, string] | null>(null);
  const lca = selected ? findLCA(root, selected[0], selected[1]) : null;
  const highlightIds = new Set<string>();

  if (lca && selected) {
    pathToNode(root, selected[0])?.forEach((p) => highlightIds.add(p.id));
    pathToNode(root, selected[1])?.forEach((p) => highlightIds.add(p.id));
  }

  return (
    <div>
      <p>LCA: {lca?.name ?? "Select two people"}</p>
      <PersonNode
        person={root}
        highlightIds={highlightIds}
        onSelect={(id) =>
          setSelected((prev) =>
            !prev ? [id, id] : prev[0] === id ? null : [prev[0], id]
          )
        }
      />
    </div>
  );
}

function PersonNode({ person, highlightIds, onSelect }: Props) {
  return (
    <div style={{ marginLeft: 16, background: highlightIds.has(person.id) ? "#fef3c7" : "transparent" }}>
      <button onClick={() => onSelect(person.id)}>{person.name}</button>
      {person.children.map((c) => (
        <PersonNode key={c.id} person={c} highlightIds={highlightIds} onSelect={onSelect} />
      ))}
    </div>
  );
}
```

---

## 4. Message Thread LCA — RN

```tsx
type Message = { id: string; text: string; parentId: string | null };

function findMessageLCA(messages: Message[], idA: string, idB: string): string | null {
  const parent = new Map(messages.map((m) => [m.id, m.parentId]));
  const ancestorsA = new Set<string>();
  let cur: string | null = idA;
  while (cur) { ancestorsA.add(cur); cur = parent.get(cur) ?? null; }

  cur = idB;
  while (cur) {
    if (ancestorsA.has(cur)) return cur;
    cur = parent.get(cur) ?? null;
  }
  return null;
}

function JumpToForkButton({ messages, replyA, replyB }: Props) {
  const forkId = findMessageLCA(messages, replyA, replyB);
  const scrollRef = useRef<FlatList>(null);

  return (
    <Pressable
      onPress={() => {
        const index = messages.findIndex((m) => m.id === forkId);
        scrollRef.current?.scrollToIndex({ index, animated: true });
      }}
    >
      <Text style={{ color: "#007AFF" }}>Jump to conversation fork</Text>
    </Pressable>
  );
}
```

---

## 5. Interview Checklist

- [ ] LCA algorithm (tree or parent map)
- [ ] Highlight path from root to LCA
- [ ] Two-click selection UX
- [ ] RN scroll to fork message

---

*End of Day 33 machine coding*
