# Day 34 — Machine Coding Revision

**React:** Tree checkboxes · select subtree · cascade select  
**React Native:** Category filter tree · multi-select filter chips

---

## Table of Contents

1. [Checkbox Tree State Model](#1-checkbox-tree-state-model)
2. [Select Subtree Logic](#2-select-subtree-logic)
3. [Indeterminate Parent State — React](#3-indeterminate-parent-state--react)
4. [Category Filter Tree — RN](#4-category-filter-tree--rn)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Checkbox Tree State Model

```ts
type CheckState = "checked" | "unchecked" | "indeterminate";

type Category = {
  id: string;
  label: string;
  children: Category[];
};

function getAllIds(node: Category): string[] {
  return [node.id, ...node.children.flatMap(getAllIds)];
}
```

---

## 2. Select Subtree Logic

```ts
function toggleSubtree(
  selected: Set<string>,
  node: Category,
  checked: boolean
): Set<string> {
  const next = new Set(selected);
  const ids = getAllIds(node);
  ids.forEach((id) => (checked ? next.add(id) : next.delete(id)));
  return next;
}

function getNodeState(node: Category, selected: Set<string>): CheckState {
  const ids = getAllIds(node);
  const checkedCount = ids.filter((id) => selected.has(id)).length;
  if (checkedCount === 0) return "unchecked";
  if (checkedCount === ids.length) return "checked";
  return "indeterminate";
}
```

---

## 3. Indeterminate Parent State — React

```tsx
function CheckboxTreeNode({
  node,
  selected,
  onToggle,
}: {
  node: Category;
  selected: Set<string>;
  onToggle: (node: Category, checked: boolean) => void;
}) {
  const state = getNodeState(node, selected);
  const ref = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (ref.current) ref.current.indeterminate = state === "indeterminate";
  }, [state]);

  return (
    <div style={{ marginLeft: 16 }}>
      <label>
        <input
          ref={ref}
          type="checkbox"
          checked={state === "checked"}
          onChange={(e) => onToggle(node, e.target.checked)}
        />
        {node.label}
      </label>
      {node.children.map((c) => (
        <CheckboxTreeNode key={c.id} node={c} selected={selected} onToggle={onToggle} />
      ))}
    </div>
  );
}

export function CategoryFilter({ root }: { root: Category }) {
  const [selected, setSelected] = useState<Set<string>>(new Set());

  const handleToggle = (node: Category, checked: boolean) => {
    setSelected((prev) => toggleSubtree(prev, node, checked));
  };

  return (
    <div>
      <CheckboxTreeNode node={root} selected={selected} onToggle={handleToggle} />
      <p>Selected: {[...selected].join(", ")}</p>
    </div>
  );
}
```

---

## 4. Category Filter Tree — RN

```tsx
import { Pressable, Text, View } from "react-native";

function CategoryRow({ node, selected, onToggle, depth }: Props) {
  const state = getNodeState(node, selected);
  const icon = state === "checked" ? "☑" : state === "indeterminate" ? "◩" : "☐";

  return (
    <View>
      <Pressable
        style={{ flexDirection: "row", paddingLeft: depth * 16, paddingVertical: 8 }}
        onPress={() => onToggle(node, state !== "checked")}
      >
        <Text>{icon} </Text>
        <Text>{node.label}</Text>
      </Pressable>
      {node.children.map((c) => (
        <CategoryRow key={c.id} node={c} selected={selected} onToggle={onToggle} depth={depth + 1} />
      ))}
    </View>
  );
}

function FilterChips({ selected, labels }: { selected: Set<string>; labels: Map<string, string> }) {
  return (
    <View style={{ flexDirection: "row", flexWrap: "wrap", gap: 8, padding: 8 }}>
      {[...selected].map((id) => (
        <View key={id} style={{ backgroundColor: "#e0e7ff", padding: 4, borderRadius: 12 }}>
          <Text>{labels.get(id)}</Text>
        </View>
      ))}
    </View>
  );
}
```

---

## 5. Interview Checklist

- [ ] Subtree select/deselect all descendants
- [ ] Indeterminate state for partial selection
- [ ] Set-based selected IDs
- [ ] RN Pressable + visual states

---

*End of Day 34 machine coding*
