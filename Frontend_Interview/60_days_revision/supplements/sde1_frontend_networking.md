# SDE 1–2 Supplement — Frontend Networking

**Topics:** HTTP · Status codes · CORS · Caching · AbortController · Retry · WebSockets · SSE

**Related days:** Day 3 (Promises), Day 5 (debounced fetch + cancel)

---

## 1. HTTP Methods & Idempotency

| Method | Purpose | Idempotent | Body |
|--------|---------|------------|------|
| GET | Read | Yes | No |
| POST | Create | No | Yes |
| PUT | Replace | Yes | Yes |
| PATCH | Partial update | Usually | Yes |
| DELETE | Remove | Yes | Optional |

---

## 2. Status Codes (must know)

| Code | Meaning | Frontend action |
|------|---------|-----------------|
| 200 | OK | Render data |
| 201 | Created | Redirect / show success |
| 204 | No content | DELETE success |
| 400 | Bad request | Show validation errors |
| 401 | Unauthorized | Redirect to login |
| 403 | Forbidden | Show permission error |
| 404 | Not found | Empty state |
| 409 | Conflict | Show duplicate/conflict message |
| 429 | Rate limited | Retry after delay |
| 500 | Server error | Generic error + retry |

---

## 3. CORS (Cross-Origin Resource Sharing)

Browser blocks JS from reading cross-origin responses unless server allows.

```text
Browser (https://app.com) → fetch https://api.other.com
Server must respond:
  Access-Control-Allow-Origin: https://app.com
  Access-Control-Allow-Credentials: true  (if cookies)
```

**Preflight:** For non-simple requests (JSON POST, custom headers), browser sends `OPTIONS` first.

```ts
fetch("https://api.example.com/data", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  credentials: "include", // send cookies — server must allow origin + credentials
  body: JSON.stringify({ name: "Alice" }),
});
```

| Term | Meaning |
|------|---------|
| Simple request | GET, POST (form), few headers — no preflight |
| Preflight | OPTIONS check before actual request |
| Same-origin | No CORS needed |

**Interview:** CORS is browser enforcement — Postman/curl bypass it. Fix on **server** (or dev proxy).

---

## 4. Caching

```http
Cache-Control: max-age=3600, public
ETag: "abc123"
```

```ts
// Conditional request — server returns 304 Not Modified if unchanged
fetch(url, { headers: { "If-None-Match": etag } });
```

| Strategy | Use |
|----------|-----|
| Browser cache | Static assets with hash in filename (`app.a1b2c3.js`) |
| CDN | Edge cache for images, JS, CSS |
| SWR/React Query | Client stale-while-revalidate |
| Service Worker | Offline cache (PWA) |

---

## 5. AbortController (Request Cancellation)

```ts
function useSearch(query: string) {
  const [results, setResults] = useState<Item[]>([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query.trim()) { setResults([]); return; }

    const controller = new AbortController();
    setLoading(true);

    fetch(`/api/search?q=${encodeURIComponent(query)}`, {
      signal: controller.signal,
    })
      .then((r) => r.json())
      .then(setResults)
      .catch((err) => {
        if (err.name !== "AbortError") console.error(err);
      })
      .finally(() => setLoading(false));

    return () => controller.abort(); // cancel on query change or unmount
  }, [query]);

  return { results, loading };
}
```

---

## 6. Retry with Exponential Backoff

```ts
async function fetchWithRetry(url: string, retries = 3): Promise<Response> {
  for (let i = 0; i < retries; i++) {
    try {
      const res = await fetch(url);
      if (res.ok || res.status < 500) return res;
    } catch (e) {
      if (i === retries - 1) throw e;
    }
    await new Promise((r) => setTimeout(r, 2 ** i * 300));
  }
  throw new Error("Max retries");
}
```

Only retry **idempotent** requests (GET) or POST with **idempotency key**.

---

## 7. WebSockets & SSE

### WebSocket (bidirectional)

```ts
const ws = new WebSocket("wss://api.example.com/chat");
ws.onmessage = (e) => setMessages((m) => [...m, JSON.parse(e.data)]);
ws.send(JSON.stringify({ type: "msg", text: "hello" }));
// cleanup: ws.close()
```

### SSE (server → client only)

```ts
const es = new EventSource("/api/events");
es.onmessage = (e) => console.log(e.data);
// Auto-reconnect built-in
```

| | WebSocket | SSE |
|---|-----------|-----|
| Direction | Both | Server → client |
| Protocol | ws:// | HTTP |
| Use | Chat, games | Live feeds, notifications |

---

## 8. REST vs GraphQL (brief)

| REST | GraphQL |
|------|---------|
| Multiple endpoints | Single `/graphql` |
| Fixed response shape | Client picks fields |
| Over-fetching common | Precise queries |
| HTTP caching easy | Needs client cache (Apollo/RQ) |

---

## Interview Q&A

**Q: Why 401 vs 403?**  
A: 401 = not authenticated. 403 = authenticated but not allowed.

**Q: How to fix CORS in dev?**  
A: Vite/webpack proxy, or server adds `Access-Control-Allow-Origin`.

**Q: Race condition in search?**  
A: AbortController or ignore stale responses with request id.

---

*End of Frontend Networking Supplement*
