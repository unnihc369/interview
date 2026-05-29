# SDE 1–2 Supplement — Testing (Jest + React Testing Library)

**Topics:** Unit tests · Component tests · Hook tests · Mock fetch · userEvent

**Related days:** Day 60 (checklist), Day 28 (mock prep)

---

## 1. Testing Pyramid (Frontend)

```text
        E2E (few)        — Cypress, Playwright
       Integration       — RTL + MSW
      Unit (many)        — Pure functions, hooks, reducers
```

**Interview default:** Focus on **React Testing Library** — test behavior, not implementation.

---

## 2. Setup (Vite + Vitest or CRA Jest)

```tsx
// vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: { environment: "jsdom", setupFiles: "./src/test/setup.ts" },
});
```

```ts
// setup.ts
import "@testing-library/jest-dom";
```

---

## 3. Component Test Example

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Counter } from "./Counter";

test("increments count on click", async () => {
  const user = userEvent.setup();
  render(<Counter />);

  expect(screen.getByRole("button", { name: /increment/i })).toBeInTheDocument();
  await user.click(screen.getByRole("button", { name: /increment/i }));

  expect(screen.getByText("Count: 1")).toBeInTheDocument();
});
```

### Query priority (RTL)

1. `getByRole` (best — a11y aligned)
2. `getByLabelText`
3. `getByPlaceholderText`
4. `getByText`
5. `getByTestId` (last resort)

---

## 4. Testing Async & Fetch

```tsx
import { render, screen, waitFor } from "@testing-library/react";
import { SearchBox } from "./SearchBox";

beforeEach(() => {
  global.fetch = vi.fn().mockResolvedValue({
    ok: true,
    json: async () => [{ id: "1", name: "Apple" }],
  });
});

test("shows search results", async () => {
  render(<SearchBox />);
  await userEvent.type(screen.getByRole("textbox"), "app");

  await waitFor(() => {
    expect(screen.getByText("Apple")).toBeInTheDocument();
  });

  expect(fetch).toHaveBeenCalledWith(expect.stringContaining("q=app"), expect.any(Object));
});
```

**MSW (Mock Service Worker):** Intercept network at service worker level — closer to real API.

---

## 5. Testing Custom Hooks

```tsx
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "./useCounter";

test("useCounter increments", () => {
  const { result } = renderHook(() => useCounter(0));

  act(() => result.current.increment());

  expect(result.current.count).toBe(1);
});
```

---

## 6. What to Test in Machine Coding Interviews

| Test | Example |
|------|---------|
| Renders initial state | Empty list message |
| User interaction | Click adds item |
| Edge case | Empty input ignored |
| Async | Loading spinner → results |
| Error | API fail shows error |

**Don't test:** Internal state variable names, implementation details.

---

## 7. userEvent vs fireEvent

| | userEvent | fireEvent |
|---|-----------|-----------|
| Realism | Simulates full interaction | Dispatches single event |
| Prefer | ✅ Default | Low-level only |

---

## Interview Q&A

**Q: Why RTL over Enzyme?**  
A: RTL encourages testing from user perspective; Enzyme tested implementation.

**Q: How to test useEffect fetch?**  
A: Mock fetch/MSW + `waitFor` + assert DOM outcome.

**Q: Snapshot tests?**  
A: OK for stable UI; brittle for frequent changes — prefer role/text queries.

---

*End of Testing Supplement*
