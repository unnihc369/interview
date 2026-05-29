# Day 40 — Machine Coding Revision

**React:** Export sorted data from tree · CSV download  
**React Native:** Sort products via BST · export share sheet

---

## Table of Contents

1. [Product BST Model](#1-product-bst-model)
2. [Export Sorted JSON/CSV — React](#2-export-sorted-jsoncsv--react)
3. [Import & Rebalance](#3-import--rebalance)
4. [RN Sort Products BST](#4-rn-sort-products-bst)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Product BST Model

```ts
type Product = { id: string; name: string; price: number };

function insertProduct(root: ProductNode | null, p: Product): ProductNode {
  if (!root) return { val: p.price, product: p, left: null, right: null };
  if (p.price < root.val) root.left = insertProduct(root.left, p);
  else root.right = insertProduct(root.right, p);
  return root;
}

function inorderProducts(root: ProductNode | null, out: Product[] = []): Product[] {
  if (!root) return out;
  inorderProducts(root.left, out);
  out.push(root.product);
  inorderProducts(root.right, out);
  return out;
}
```

---

## 2. Export Sorted JSON/CSV — React

```tsx
function ExportPanel({ root }: { root: ProductNode | null }) {
  const sorted = useMemo(() => inorderProducts(root), [root]);

  const exportJSON = () => {
    const blob = new Blob([JSON.stringify(sorted, null, 2)], { type: "application/json" });
    downloadBlob(blob, "products-sorted.json");
  };

  const exportCSV = () => {
    const header = "id,name,price\n";
    const rows = sorted.map((p) => `${p.id},${p.name},${p.price}`).join("\n");
    const blob = new Blob([header + rows], { type: "text/csv" });
    downloadBlob(blob, "products-sorted.csv");
  };

  return (
    <div style={{ display: "flex", gap: 8 }}>
      <button onClick={exportJSON}>Export JSON</button>
      <button onClick={exportCSV}>Export CSV</button>
      <span>{sorted.length} products (price sorted)</span>
    </div>
  );
}

function downloadBlob(blob: Blob, filename: string) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}
```

---

## 3. Import & Rebalance

```tsx
function importAndBalance(json: string): ProductNode | null {
  const products: Product[] = JSON.parse(json);
  const prices = products.map((p) => p.price).sort((a, b) => a - b);
  // Rebuild balanced tree from sorted prices + map back to products
  return sortedProductsToBST(products.sort((a, b) => a.price - b.price));
}

function sortedProductsToBST(products: Product[]): ProductNode | null {
  function build(lo: number, hi: number): ProductNode | null {
    if (lo > hi) return null;
    const mid = (lo + hi) >> 1;
    const p = products[mid];
    return { val: p.price, product: p, left: build(lo, mid - 1), right: build(mid + 1, hi) };
  }
  return build(0, products.length - 1);
}
```

---

## 4. RN Sort Products BST

```tsx
import * as Sharing from "expo-sharing";
import * as FileSystem from "expo-file-system";

function ProductListRN({ root }: { root: ProductNode | null }) {
  const sorted = useMemo(() => inorderProducts(root), [root]);

  const shareExport = async () => {
    const csv = sorted.map((p) => `${p.name},${p.price}`).join("\n");
    const path = FileSystem.cacheDirectory + "products.csv";
    await FileSystem.writeAsStringAsync(path, csv);
    await Sharing.shareAsync(path);
  };

  return (
    <View style={{ flex: 1 }}>
      <FlatList
        data={sorted}
        keyExtractor={(p) => p.id}
        renderItem={({ item }) => (
          <View style={{ padding: 12, flexDirection: "row", justifyContent: "space-between" }}>
            <Text>{item.name}</Text>
            <Text>${item.price.toFixed(2)}</Text>
          </View>
        )}
      />
      <Button title="Share Sorted CSV" onPress={shareExport} />
    </View>
  );
}
```

---

## 5. Interview Checklist

- [ ] Inorder → sorted export
- [ ] CSV/JSON download (web) or share (RN)
- [ ] Rebuild balanced BST from sorted import
- [ ] Explain O(n) export vs O(n log n) sort-then-build

---

*End of Day 40 machine coding*
