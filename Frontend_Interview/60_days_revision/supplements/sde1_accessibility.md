# SDE 1–2 Supplement — Accessibility (a11y)

**Topics:** Semantic HTML · ARIA · Keyboard navigation · Focus management · Screen readers

**Related days:** Day 14 (mention a11y), Machine coding classics (Modal, Dropdown)

---

## 1. Why Accessibility Matters

- Legal compliance (WCAG 2.1 AA)
- Better UX for everyone (keyboard, mobile, low vision)
- **Interview signal:** Shows production-ready thinking

---

## 2. Semantic HTML First

```tsx
// ❌ div soup
<div onClick={submit}>Submit</div>

// ✅ semantic
<button type="submit">Submit</button>

<nav aria-label="Main">...</nav>
<main>...</main>
<header>...</header>
<footer>...</footer>
<article>...</article>
```

| Element | Use |
|---------|-----|
| `<button>` | Actions |
| `<a href>` | Navigation |
| `<label htmlFor>` | Form labels |
| `<h1>`–`<h6>` | Heading hierarchy |
| `<ul>/<li>` | Lists |

**Rule:** Don't use ARIA when native HTML works.

---

## 3. ARIA Essentials

```tsx
// Labels
<button aria-label="Close dialog">×</button>
<input aria-describedby="email-hint" />
<p id="email-hint">We'll never share your email</p>

// Live regions — dynamic updates
<div role="status" aria-live="polite">{statusMessage}</div>
<div role="alert" aria-live="assertive">{errorMessage}</div>

// States
<button aria-expanded={open} aria-controls="menu-id">Menu</button>
<ul id="menu-id" role="menu" hidden={!open}>...</ul>

// Modal
<div role="dialog" aria-modal="true" aria-labelledby="title-id">
```

| Attribute | Purpose |
|-----------|---------|
| `aria-label` | Accessible name when no visible text |
| `aria-labelledby` | Reference another element's text |
| `aria-describedby` | Extra description |
| `aria-hidden="true"` | Hide decorative icons from screen readers |
| `aria-live` | Announce dynamic changes |

---

## 4. Keyboard Navigation

| Key | Expected behavior |
|-----|-------------------|
| Tab / Shift+Tab | Move focus |
| Enter / Space | Activate button |
| Escape | Close modal/dropdown |
| Arrow keys | Navigate menu/list/tabs |
| Home / End | First/last item in list |

```tsx
function Menu({ items }: { items: string[] }) {
  const [focusIndex, setFocusIndex] = useState(0);

  const onKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === "ArrowDown") {
      e.preventDefault();
      setFocusIndex((i) => Math.min(i + 1, items.length - 1));
    }
    if (e.key === "ArrowUp") {
      e.preventDefault();
      setFocusIndex((i) => Math.max(i - 1, 0));
    }
  };

  return (
    <ul role="listbox" onKeyDown={onKeyDown}>
      {items.map((item, i) => (
        <li key={item} role="option" tabIndex={i === focusIndex ? 0 : -1} aria-selected={i === focusIndex}>
          {item}
        </li>
      ))}
    </ul>
  );
}
```

---

## 5. Focus Management

```tsx
// Modal: trap focus inside, restore on close (see machine_coding_classics)

// Skip link for keyboard users
<a href="#main-content" className="skip-link">Skip to main content</a>

// Visible focus styles — never remove outline without replacement
button:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

---

## 6. Forms Accessibility

```tsx
<label htmlFor="email">Email</label>
<input
  id="email"
  type="email"
  aria-invalid={!!errors.email}
  aria-describedby={errors.email ? "email-error" : undefined}
/>
{errors.email && (
  <span id="email-error" role="alert">{errors.email}</span>
)}
```

- Associate labels with `htmlFor` + `id`
- Group related fields with `<fieldset>` + `<legend>`
- Announce errors with `role="alert"`

---

## 7. Testing Accessibility

```tsx
import { axe, toHaveNoViolations } from "jest-axe";
expect.extend(toHaveNoViolations);

test("no a11y violations", async () => {
  const { container } = render(<MyComponent />);
  expect(await axe(container)).toHaveNoViolations();
});
```

**Manual checks:** Tab through entire flow. Test with VoiceOver (Mac) or NVDA (Windows).

---

## Interview Q&A

**Q: aria-label vs aria-labelledby?**  
A: label sets name directly; labelledby points to existing element id.

**Q: When is div+onClick OK?**  
A: Almost never for actions — use `<button>`. div+click fails keyboard and screen readers.

**Q: How to make custom dropdown accessible?**  
A: role="listbox"/"option", aria-expanded, keyboard arrows, focus management.

---

*End of Accessibility Supplement*
