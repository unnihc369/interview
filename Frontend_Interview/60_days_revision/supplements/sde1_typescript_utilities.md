# SDE 1–2 Supplement — TypeScript Utility Types

**Topics:** Partial · Pick · Omit · Required · Record · Exclude · Extract · Template literals · `satisfies`

**Related days:** Day 2 (union/intersection), Day 4 (mapped types, keyof), Day 5 (conditional types)

---

## 1. Built-in Utility Types

```ts
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Partial — all optional (updates, patches)
type UserUpdate = Partial<User>;
// { id?: string; name?: string; ... }

// Required — all required
type RequiredUser = Required<Partial<User>>;

// Pick — select keys
type UserPreview = Pick<User, "id" | "name">;
// { id: string; name: string }

// Omit — exclude keys
type UserWithoutEmail = Omit<User, "email">;
// { id: string; name: string; age: number }

// Record — object with key type K and value type V
type RolePermissions = Record<"admin" | "user", string[]>;
// { admin: string[]; user: string[] }
```

---

## 2. Exclude & Extract

```ts
type All = "click" | "focus" | "blur" | "submit";
type MouseEvents = Exclude<All, "focus" | "blur" | "submit">; // "click"
type FocusEvents = Extract<All, "focus" | "blur">;             // "focus" | "blur"
```

Works on union types — common with discriminated unions.

---

## 3. ReturnType & Parameters

```ts
async function fetchUser(id: string) {
  return { id, name: "Alice" };
}

type FetchUserReturn = Awaited<ReturnType<typeof fetchUser>>;
// { id: string; name: string }

type FetchUserParams = Parameters<typeof fetchUser>; // [string]
```

---

## 4. Discriminated Unions

```ts
type ApiState =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User[] }
  | { status: "error"; error: string };

function render(state: ApiState) {
  switch (state.status) {
    case "idle": return null;
    case "loading": return <Spinner />;
    case "success": return state.data.map(...); // data narrowed
    case "error": return <Error msg={state.error} />;
  }
}
```

---

## 5. Template Literal Types

```ts
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type ApiRoute = `/api/${string}`;

type EventName = "click" | "focus";
type HandlerName = `on${Capitalize<EventName>}`; // "onClick" | "onFocus"

// CSS-in-JS
type Spacing = "sm" | "md" | "lg";
type MarginClass = `m-${Spacing}`; // "m-sm" | "m-md" | "m-lg"
```

---

## 6. `satisfies` Operator (TS 4.9+)

Preserves **literal types** while checking shape:

```ts
// Without satisfies — type widened to string
const config = {
  theme: "dark",
  apiUrl: "https://api.example.com",
};

// With satisfies — literals preserved + validation
const config = {
  theme: "dark",
  apiUrl: "https://api.example.com",
} satisfies Record<string, string>;

config.theme; // "dark" (not string)
```

---

## 7. Practical React Patterns

```ts
// Form with typed fields
type FormData = { email: string; password: string };

function handleChange<K extends keyof FormData>(key: K, value: FormData[K]) {
  setForm((prev) => ({ ...prev, [key]: value }));
}

// API response wrapper
type ApiResponse<T> =
  | { ok: true; data: T }
  | { ok: false; error: string };

// Component props from interface
type ButtonProps = Pick<React.ComponentProps<"button">, "onClick" | "disabled"> & {
  variant: "primary" | "secondary";
};
```

---

## Cheat Sheet

| Utility | Result |
|---------|--------|
| `Partial<T>` | All props optional |
| `Required<T>` | All props required |
| `Pick<T, K>` | Subset of keys |
| `Omit<T, K>` | Remove keys |
| `Record<K, V>` | Key-value map type |
| `Exclude<U, E>` | Remove from union |
| `Extract<U, E>` | Keep in union |
| `ReturnType<F>` | Function return type |
| `satisfies T` | Validate without widening |

---

*End of TypeScript Utilities Supplement*
