# SDE 1–2 Supplement — State Management

**Topics:** Context · Prop drilling · Redux Toolkit · Zustand · TanStack Query · Optimistic updates

**Related days:** Day 1 (callback params vs global state), Day 3 (useQueue)

---

## 1. When to Use What

| Solution | Best for |
|----------|----------|
| `useState` / `useReducer` | Local UI state |
| Context | Theme, auth user, locale — low-frequency updates |
| Redux / Zustand | Complex global client state |
| TanStack Query / SWR | **Server state** (fetch, cache, sync) |
| URL state | Filters, pagination, shareable views |

**Rule:** Server state ≠ client state. Don't put API cache in Redux if React Query handles it.

---

## 2. Context Pattern

```tsx
type AuthContextType = { user: User | null; login: (u: User) => void; logout: () => void };

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const value = useMemo(
    () => ({
      user,
      login: setUser,
      logout: () => setUser(null),
    }),
    [user]
  );
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth outside AuthProvider");
  return ctx;
}
```

**Pitfall:** One big context → all consumers re-render on any change. Split contexts or use selectors (Zustand).

---

## 3. Redux Toolkit (RTK)

```ts
// store/slices/cartSlice.ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

type CartItem = { id: string; qty: number };

const cartSlice = createSlice({
  name: "cart",
  initialState: { items: [] as CartItem[] },
  reducers: {
    addItem(state, action: PayloadAction<CartItem>) {
      const existing = state.items.find((i) => i.id === action.payload.id);
      if (existing) existing.qty += action.payload.qty;
      else state.items.push(action.payload);
    },
    removeItem(state, action: PayloadAction<string>) {
      state.items = state.items.filter((i) => i.id !== action.payload);
    },
  },
});

export const { addItem, removeItem } = cartSlice.actions;
export default cartSlice.reducer;
```

```tsx
// Component
const items = useSelector((state: RootState) => state.cart.items);
const dispatch = useDispatch();
dispatch(addItem({ id: "1", qty: 1 }));
```

**RTK Query:** Built-in data fetching layer on top of Redux.

---

## 4. Zustand (lightweight global store)

```ts
import { create } from "zustand";

type Store = {
  count: number;
  increment: () => void;
};

export const useStore = create<Store>((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
}));

// Component — only re-renders when count changes
const count = useStore((s) => s.count);
```

Less boilerplate than Redux — popular for SDE 1–2 machine coding.

---

## 5. TanStack Query (React Query)

```tsx
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then((r) => r.json()),
    staleTime: 60_000, // fresh for 1 min
  });

  if (isLoading) return <Spinner />;
  if (error) return <Error />;

  return data.map((u) => <UserRow key={u.id} user={u} />);
}

function useDeleteUser() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (id: string) => fetch(`/api/users/${id}`, { method: "DELETE" }),
    onSuccess: () => qc.invalidateQueries({ queryKey: ["users"] }),
  });
}
```

| Concept | Meaning |
|---------|---------|
| `queryKey` | Cache identity |
| `staleTime` | How long data is "fresh" |
| `invalidateQueries` | Refetch after mutation |
| `placeholderData` | Show old data while refetching |

---

## 6. Optimistic Updates

```tsx
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    await qc.cancelQueries({ queryKey: ["todos"] });
    const previous = qc.getQueryData(["todos"]);
    qc.setQueryData(["todos"], (old: Todo[]) => [...old, newTodo]);
    return { previous };
  },
  onError: (_err, _new, context) => {
    qc.setQueryData(["todos"], context?.previous);
  },
  onSettled: () => qc.invalidateQueries({ queryKey: ["todos"] }),
});
```

---

## Interview Q&A

**Q: Context vs Redux?**  
A: Context for simple shared values; Redux/Zustand when many updates, middleware, devtools, complex logic.

**Q: Where to put form state?**  
A: Local unless multi-step wizard spans routes — then URL or context.

**Q: React Query vs Redux for API data?**  
A: React Query — caching, dedup, background refetch built-in.

---

*End of State Management Supplement*
