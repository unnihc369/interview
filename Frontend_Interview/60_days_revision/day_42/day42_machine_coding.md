# Day 42 — Machine Coding Revision

**React:** BST range sum widget · dual-handle slider filter  
**React Native:** Range slider filter · price range on product BST

---

## Table of Contents

1. [Range Sum on BST](#1-range-sum-on-bst)
2. [Range Slider Widget — React](#2-range-slider-widget--react)
3. [Live Sum Display](#3-live-sum-display)
4. [RN Range Slider Filter](#4-rn-range-slider-filter)
5. [Week 6 Revision Checklist](#5-week-6-revision-checklist)

---

## 1. Range Sum on BST

```ts
function rangeSumBST(root: TreeNode | null, low: number, high: number): number {
  if (!root) return 0;
  let sum = 0;
  if (root.val >= low && root.val <= high) sum += root.val;
  if (root.val > low) sum += rangeSumBST(root.left, low, high);
  if (root.val < high) sum += rangeSumBST(root.right, low, high);
  return sum;
}

function nodesInRange(root: TreeNode | null, low: number, high: number): TreeNode[] {
  const result: TreeNode[] = [];
  function dfs(node: TreeNode | null) {
    if (!node) return;
    if (node.val > low) dfs(node.left);
    if (node.val >= low && node.val <= high) result.push(node);
    if (node.val < high) dfs(node.right);
  }
  dfs(root);
  return result;
}
```

**Pruning:** Skip left if `val <= low`; skip right if `val >= high`.

---

## 2. Range Slider Widget — React

```tsx
function RangeSumWidget({ root, minPrice, maxPrice }: Props) {
  const [low, setLow] = useState(minPrice);
  const [high, setHigh] = useState(maxPrice);

  const sum = useMemo(() => rangeSumBST(root, low, high), [root, low, high]);
  const items = useMemo(() => nodesInRange(root, low, high), [root, low, high]);

  return (
    <div style={{ padding: 16 }}>
      <h3>Price Range Filter</h3>

      <label>
        Min: ${low}
        <input
          type="range"
          min={minPrice}
          max={maxPrice}
          value={low}
          onChange={(e) => setLow(Math.min(+e.target.value, high))}
        />
      </label>

      <label>
        Max: ${high}
        <input
          type="range"
          min={minPrice}
          max={maxPrice}
          value={high}
          onChange={(e) => setHigh(Math.max(+e.target.value, low))}
        />
      </label>

      <p><strong>Range sum:</strong> ${sum.toFixed(2)}</p>
      <p><strong>Products in range:</strong> {items.length}</p>

      <ul>
        {items.map((n) => (
          <li key={n.val}>${n.val}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 3. Live Sum Display

Debounce slider for large trees (optional):

```tsx
const debouncedLow = useDebounce(low, 150);
const debouncedHigh = useDebounce(high, 150);
const sum = useMemo(() => rangeSumBST(root, debouncedLow, debouncedHigh), [root, debouncedLow, debouncedHigh]);
```

---

## 4. RN Range Slider Filter

```tsx
import Slider from "@react-native-community/slider";

function PriceRangeFilterRN({ root }: { root: TreeNode | null }) {
  const [low, setLow] = useState(0);
  const [high, setHigh] = useState(1000);
  const sum = useMemo(() => rangeSumBST(root, low, high), [root, low, high]);
  const products = useMemo(() => nodesInRange(root, low, high), [root, low, high]);

  return (
    <View style={{ padding: 16 }}>
      <Text>Min: ${low.toFixed(0)}</Text>
      <Slider minimumValue={0} maximumValue={1000} value={low} onValueChange={(v) => setLow(Math.min(v, high))} />
      <Text>Max: ${high.toFixed(0)}</Text>
      <Slider minimumValue={0} maximumValue={1000} value={high} onValueChange={(v) => setHigh(Math.max(v, low))} />
      <Text style={{ fontSize: 18, fontWeight: "700", marginTop: 12 }}>Total: ${sum.toFixed(2)}</Text>
      <FlatList
        data={products}
        keyExtractor={(n) => String(n.val)}
        renderItem={({ item }) => <Text style={{ padding: 8 }}>${item.val}</Text>}
      />
    </View>
  );
}
```

---

## 5. Week 6 Revision Checklist

- [ ] BST search/insert/delete
- [ ] Validate with bounds
- [ ] Kth smallest / top-k leaderboard
- [ ] Export sorted / import balanced
- [ ] Range sum with slider filter
- [ ] RN draggable / share export

---

*End of Day 42 machine coding*
