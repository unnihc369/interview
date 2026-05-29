# Day 39 — Machine Coding Revision

**React:** Leaderboard kth rank widget · top-k display  
**React Native:** Top scores screen · pagination by rank

---

## Table of Contents

1. [Score BST Model](#1-score-bst-model)
2. [Leaderboard Widget — React](#2-leaderboard-widget--react)
3. [Top-K Hook](#3-top-k-hook)
4. [RN Top Scores Screen](#4-rn-top-scores-screen)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Score BST Model

```ts
type Player = { id: string; name: string; score: number };

// BST keyed by score (handle ties with id in composite key)
function insertScore(root: TreeNode | null, player: Player): TreeNode {
  if (!root) return { val: player.score, player, left: null, right: null };
  const key = player.score;
  if (key < root.val) root.left = insertScore(root.left, player);
  else root.right = insertScore(root.right, player);
  return root;
}
```

---

## 2. Leaderboard Widget — React

```tsx
function useLeaderboard(initial: Player[] = []) {
  const [root, setRoot] = useState<TreeNode | null>(null);

  useEffect(() => {
    let r: TreeNode | null = null;
    for (const p of initial) r = insertScore(r, p);
    setRoot(r);
  }, [initial]);

  const addScore = (player: Player) => setRoot((r) => insertScore(r, player));

  const topK = (k: number) => reverseInorderTopK(root, k);
  const kthRank = (k: number) => kthLargest(root, k);
  const rankOf = (score: number) => countGreaterOrEqual(root, score);

  return { addScore, topK, kthRank, rankOf };
}

function reverseInorderTopK(root: TreeNode | null, k: number): Player[] {
  const stack: TreeNode[] = [];
  let cur = root;
  const result: Player[] = [];
  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.right; }
    cur = stack.pop()!;
    result.push(cur.player);
    if (result.length === k) break;
    cur = cur.left;
  }
  return result;
}
```

```tsx
function Leaderboard() {
  const { addScore, topK, kthRank } = useLeaderboard(seedPlayers);
  const [k, setK] = useState(3);
  const top = topK(k);
  const kthScore = kthRank(k);

  return (
    <div>
      <h3>Top {k} Players</h3>
      <input type="number" value={k} onChange={(e) => setK(+e.target.value)} min={1} />
      <ol>
        {top.map((p, i) => (
          <li key={p.id}>#{i + 1} {p.name} — {p.score}</li>
        ))}
      </ol>
      <p>{k}th highest score: {kthScore?.score ?? "N/A"}</p>
    </div>
  );
}
```

---

## 3. Top-K Hook

```tsx
function useTopK(root: TreeNode | null, k: number) {
  return useMemo(() => reverseInorderTopK(root, k), [root, k]);
}
```

For live updates, recompute on root change — mention augmented tree for O(log n) if scale matters.

---

## 4. RN Top Scores Screen

```tsx
function TopScoresScreen() {
  const { topK, addScore } = useLeaderboard(players);
  const [page, setPage] = useState(1);
  const pageSize = 10;
  const allTop = reverseInorderTopK(root, page * pageSize);
  const visible = allTop.slice((page - 1) * pageSize, page * pageSize);

  return (
    <View style={{ flex: 1 }}>
      <FlatList
        data={visible}
        keyExtractor={(p) => p.id}
        renderItem={({ item, index }) => (
          <View style={{ flexDirection: "row", padding: 16, borderBottomWidth: 1 }}>
            <Text style={{ width: 40, fontWeight: "700" }}>#{(page - 1) * pageSize + index + 1}</Text>
            <Text style={{ flex: 1 }}>{item.name}</Text>
            <Text>{item.score}</Text>
          </View>
        )}
        onEndReached={() => setPage((p) => p + 1)}
      />
    </View>
  );
}
```

**Pull-to-refresh:** Rebuild BST from API scores.

---

## 5. Interview Checklist

- [ ] Kth largest via reverse inorder
- [ ] Top-k list UI
- [ ] Rank input / slider
- [ ] RN FlatList pagination

---

*End of Day 39 machine coding*
