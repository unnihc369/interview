# SDE 1–2 Supplement — SSR, SSG & Next.js Basics

**Topics:** CSR vs SSR vs SSG · Hydration · Next.js App Router overview

**Related days:** Day 60 (full stack project)

---

## 1. Rendering Strategies

| Strategy | When HTML is built | First paint | SEO | Use case |
|----------|-------------------|-------------|-----|----------|
| **CSR** | Browser (JS bundle) | Slow — blank then JS | Poor | Dashboards, auth apps |
| **SSR** | Server per request | Fast | Good | Personalized pages |
| **SSG** | Build time | Fast (CDN) | Good | Blogs, marketing |
| **ISR** | SSG + revalidate | Fast + fresh | Good | E-commerce catalog |

```text
CSR:  Browser → empty HTML → download JS → render → paint
SSR:  Browser → server renders HTML → paint → hydrate JS
SSG:  Build → static HTML on CDN → hydrate on client
```

---

## 2. Hydration

Server sends HTML + React attaches event listeners and state on client.

**Hydration mismatch:** Server HTML ≠ client first render → React warning.

```tsx
// ❌ Causes mismatch — Date differs server vs client
<div>{new Date().toLocaleString()}</div>

// ✅ Render after mount
const [time, setTime] = useState<string | null>(null);
useEffect(() => setTime(new Date().toLocaleString()), []);
return <div>{time ?? "..."}</div>;
```

---

## 3. Next.js App Router (v13+)

```
app/
  layout.tsx      ← shared shell
  page.tsx        ← route UI (default Server Component)
  loading.tsx     ← Suspense fallback
  error.tsx       ← Error boundary
  api/users/route.ts  ← Route Handler (API)
```

### Server vs Client Components

```tsx
// Server Component (default) — no useState/useEffect, can fetch DB directly
async function UserList() {
  const users = await db.user.findMany();
  return users.map((u) => <li key={u.id}>{u.name}</li>);
}

// Client Component — interactivity
"use client";
import { useState } from "react";
export function Counter() {
  const [n, setN] = useState(0);
  return <button onClick={() => setN(n + 1)}>{n}</button>;
}
```

| Server Component | Client Component |
|------------------|------------------|
| Fetch on server | useState, useEffect |
| No JS shipped for static parts | Event handlers |
| Can't use browser APIs | Full React hooks |

---

## 4. Data Fetching in Next.js

```tsx
// Server Component — direct async
async function Page() {
  const res = await fetch("https://api.example.com/posts", {
    next: { revalidate: 60 }, // ISR: revalidate every 60s
  });
  const posts = await res.json();
  return <PostList posts={posts} />;
}
```

---

## Interview Q&A

**Q: When SSR over CSR?**  
A: SEO-critical pages, slow devices, first-contentful-paint matters.

**Q: Hydration cost?**  
A: Server HTML shows fast but client still downloads + runs JS to make interactive.

**Q: Server Component benefit?**  
A: Less client JS, direct backend access, better security (secrets stay on server).

---

*End of SSR & Next.js Supplement*
