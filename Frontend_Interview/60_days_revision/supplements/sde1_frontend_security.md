# SDE 1–2 Supplement — Frontend Security

**Topics:** XSS · CSRF · CSP · Token storage · Same-origin · Sanitization

**Related days:** Day 5 (debounced fetch), Day 43 (autocomplete user input)

---

## Table of Contents

1. [Same-Origin Policy](#1-same-origin-policy)
2. [XSS (Cross-Site Scripting)](#2-xss-cross-site-scripting)
3. [CSRF (Cross-Site Request Forgery)](#3-csrf-cross-site-request-forgery)
4. [Content Security Policy (CSP)](#4-content-security-policy-csp)
5. [Authentication Token Storage](#5-authentication-token-storage)
6. [React-Specific Security](#6-react-specific-security)
7. [Interview Q&A](#7-interview-qa)

---

## 1. Same-Origin Policy

**Origin** = `protocol + host + port`

```text
https://app.example.com:443/page  →  origin: https://app.example.com
```

| Comparison | Same origin? |
|------------|--------------|
| `app.com` vs `api.app.com` | No (different host) |
| `http` vs `https` | No |
| `:3000` vs `:8080` | No |

Browser blocks JS from reading cross-origin responses (drives **CORS** — see [networking supplement](sde1_frontend_networking.md)).

---

## 2. XSS (Cross-Site Scripting)

Attacker injects malicious script that runs in victim's browser.

| Type | How | Example |
|------|-----|---------|
| **Stored** | Saved in DB, served to all users | Comment with `<script>` |
| **Reflected** | In URL/query, echoed in page | `?q=<script>alert(1)</script>` |
| **DOM-based** | Client JS writes untrusted data to DOM | `innerHTML = userInput` |

### Prevention

```tsx
// ✅ React escapes by default in JSX
<div>{userComment}</div>

// ❌ NEVER with untrusted data
<div dangerouslySetInnerHTML={{ __html: userComment }} />

// ✅ If HTML required — sanitize first
import DOMPurify from "dompurify";
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userComment) }} />
```

| Defense | Detail |
|---------|--------|
| Escape output | React JSX auto-escapes |
| Sanitize HTML | DOMPurify before `dangerouslySetInnerHTML` |
| Avoid `eval`, `new Function(userInput)` | Never run user strings as code |
| HttpOnly cookies | JS can't read session cookie → limits token theft |
| CSP | Block inline scripts |

---

## 3. CSRF (Cross-Site Request Forgery)

Attacker tricks logged-in user's browser into making **unwanted authenticated requests**.

```text
User logged into bank.com (cookie sent automatically)
Attacker site: <img src="https://bank.com/transfer?to=attacker&amt=1000">
Browser sends cookie → bank executes transfer
```

### Prevention

| Method | How |
|--------|-----|
| **CSRF token** | Server embeds token in form; validate on POST |
| **SameSite cookies** | `SameSite=Strict/Lax` — cookie not sent on cross-site requests |
| **Custom headers** | `X-CSRF-Token` or `Authorization: Bearer` — simple requests can't set arbitrary headers cross-origin |
| **Double-submit cookie** | Token in cookie + header must match |

```ts
// fetch with CSRF header (token from meta tag or cookie)
fetch("/api/transfer", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-CSRF-Token": getCsrfToken(),
  },
  credentials: "include",
  body: JSON.stringify(data),
});
```

**Note:** JWT in `Authorization` header (not cookie) is naturally CSRF-resistant for API calls.

---

## 4. Content Security Policy (CSP)

HTTP header telling browser which resources are allowed.

```http
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://cdn.trusted.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
```

| Directive | Controls |
|-----------|----------|
| `script-src` | JS sources — blocks inline `<script>` unless `'unsafe-inline'` |
| `connect-src` | fetch, XHR, WebSocket endpoints |
| `frame-ancestors` | Who can embed your page (clickjacking defense) |

**Interview:** CSP is defense-in-depth — even if XSS slips through, inline script may be blocked.

---

## 5. Authentication Token Storage

| Storage | XSS risk | CSRF risk | Use when |
|---------|----------|-----------|----------|
| **localStorage** | High — any JS can read | Low | Avoid for sensitive tokens |
| **sessionStorage** | High — tab scoped | Low | Short-lived non-auth data |
| **HttpOnly cookie** | Low — JS can't access | High (mitigate SameSite) | Session-based auth |
| **Memory only** | Lost on refresh | Low | SPA with short-lived access token + refresh via HttpOnly |

### Recommended SPA pattern

```text
Access token  → memory (or short-lived cookie)
Refresh token → HttpOnly, Secure, SameSite=Strict cookie
```

Never log tokens. Never put JWT in URL query params.

---

## 6. React-Specific Security

```tsx
// Validate URLs before href
const safeUrl = (url: string) => {
  try {
    const u = new URL(url);
    return ["http:", "https:"].includes(u.protocol) ? url : "#";
  } catch { return "#"; }
};

<a href={safeUrl(userLink)} rel="noopener noreferrer" target="_blank">Link</a>
```

| Risk | Mitigation |
|------|------------|
| `javascript:` URLs | Validate protocol |
| `target="_blank"` tab hijack | Always `rel="noopener noreferrer"` |
| Third-party scripts | CSP + Subresource Integrity (SRI) |
| Env secrets in bundle | Only `VITE_*` / `NEXT_PUBLIC_*` are exposed — never API secrets |

---

## 7. Interview Q&A

**Q: XSS vs CSRF?**  
A: XSS runs attacker's script in your page. CSRF forges requests using victim's existing session.

**Q: Why is localStorage bad for JWT?**  
A: Any XSS can `localStorage.getItem('token')` and exfiltrate.

**Q: Does React prevent XSS?**  
A: JSX escaping helps; `dangerouslySetInnerHTML` and URL injection bypass it.

**Q: SameSite=Lax vs Strict?**  
A: Strict never sends cookie on cross-site navigation. Lax sends on top-level GET (links).

---

## One-Line Revision

```text
XSS = sanitize/escape; CSRF = token/SameSite/header; JWT → HttpOnly cookie or memory; CSP = header whitelist.
```

---

*End of Frontend Security Supplement*
