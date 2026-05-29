# SDE 1–2 Supplement — Machine Coding Classics

**High-frequency UI components** not covered as dedicated builds in the 60-day track.

**Related days:** Day 43 (autocomplete), Day 5 (debounced search), Day 14 (tabs)

---

## Table of Contents

1. [Modal / Dialog](#1-modal--dialog)
2. [Dropdown / Select](#2-dropdown--select)
3. [Pagination](#3-pagination)
4. [Toast / Notification](#4-toast--notification)
5. [Data Table (sort + filter)](#5-data-table-sort--filter)
6. [Multi-Select Tags Input](#6-multi-select-tags-input)
7. [Star Rating](#7-star-rating)
8. [File Upload](#8-file-upload)
9. [Timer / Stopwatch](#9-timer--stopwatch)
10. [Interview Checklist](#10-interview-checklist)

---

## 1. Modal / Dialog

**Requirements:** Open/close, focus trap, Escape to close, click backdrop to close, portal to `document.body`.

```tsx
import { useEffect, useRef } from "react";
import { createPortal } from "react-dom";

function Modal({ open, onClose, title, children }: {
  open: boolean; onClose: () => void; title: string; children: React.ReactNode;
}) {
  const dialogRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!open) return;
    const prev = document.activeElement as HTMLElement;
    dialogRef.current?.focus();

    const onKey = (e: KeyboardEvent) => e.key === "Escape" && onClose();
    document.addEventListener("keydown", onKey);
    document.body.style.overflow = "hidden";

    return () => {
      document.removeEventListener("keydown", onKey);
      document.body.style.overflow = "";
      prev?.focus();
    };
  }, [open, onClose]);

  if (!open) return null;

  return createPortal(
    <div className="backdrop" onClick={onClose} role="presentation">
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        onClick={(e) => e.stopPropagation()}
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>,
    document.body
  );
}
```

---

## 2. Dropdown / Select

**Requirements:** Toggle open, click outside to close, keyboard (↑↓ Enter Escape).

```tsx
function useClickOutside(ref: React.RefObject<HTMLElement>, handler: () => void) {
  useEffect(() => {
    const listener = (e: MouseEvent) => {
      if (!ref.current?.contains(e.target as Node)) handler();
    };
    document.addEventListener("mousedown", listener);
    return () => document.removeEventListener("mousedown", listener);
  }, [ref, handler]);
}

function Dropdown({ options, value, onChange }: {
  options: { label: string; value: string }[];
  value: string;
  onChange: (v: string) => void;
}) {
  const [open, setOpen] = useState(false);
  const ref = useRef<HTMLDivElement>(null);
  useClickOutside(ref, () => setOpen(false));

  const selected = options.find((o) => o.value === value);

  return (
    <div ref={ref} className="dropdown">
      <button aria-expanded={open} onClick={() => setOpen((o) => !o)}>
        {selected?.label ?? "Select..."}
      </button>
      {open && (
        <ul role="listbox">
          {options.map((o) => (
            <li
              key={o.value}
              role="option"
              aria-selected={o.value === value}
              onClick={() => { onChange(o.value); setOpen(false); }}
            >
              {o.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## 3. Pagination

```tsx
function Pagination({ page, totalPages, onPageChange }: {
  page: number; totalPages: number; onPageChange: (p: number) => void;
}) {
  const pages = useMemo(() => {
    const delta = 2;
    const range: number[] = [];
    for (let i = Math.max(1, page - delta); i <= Math.min(totalPages, page + delta); i++) {
      range.push(i);
    }
    return range;
  }, [page, totalPages]);

  return (
    <nav aria-label="Pagination">
      <button disabled={page <= 1} onClick={() => onPageChange(page - 1)}>Prev</button>
      {pages.map((p) => (
        <button
          key={p}
          aria-current={p === page ? "page" : undefined}
          onClick={() => onPageChange(p)}
        >
          {p}
        </button>
      ))}
      <button disabled={page >= totalPages} onClick={() => onPageChange(page + 1)}>Next</button>
    </nav>
  );
}
```

---

## 4. Toast / Notification

```tsx
type Toast = { id: string; message: string; type: "success" | "error" };

const ToastContext = createContext<{
  toasts: Toast[];
  addToast: (msg: string, type?: Toast["type"]) => void;
} | null>(null);

export function ToastProvider({ children }: { children: React.ReactNode }) {
  const [toasts, setToasts] = useState<Toast[]>([]);

  const addToast = useCallback((message: string, type: Toast["type"] = "success") => {
    const id = crypto.randomUUID();
    setToasts((t) => [...t, { id, message, type }]);
    setTimeout(() => setToasts((t) => t.filter((x) => x.id !== id)), 4000);
  }, []);

  return (
    <ToastContext.Provider value={{ toasts, addToast }}>
      {children}
      <div role="region" aria-live="polite" className="toast-container">
        {toasts.map((t) => (
          <div key={t.id} className={`toast toast-${t.type}`}>{t.message}</div>
        ))}
      </div>
    </ToastContext.Provider>
  );
}
```

---

## 5. Data Table (sort + filter)

```tsx
function DataTable<T extends Record<string, unknown>>({
  data,
  columns,
}: {
  data: T[];
  columns: { key: keyof T; label: string; sortable?: boolean }[];
}) {
  const [sortKey, setSortKey] = useState<keyof T | null>(null);
  const [sortDir, setSortDir] = useState<"asc" | "desc">("asc");
  const [filter, setFilter] = useState("");

  const filtered = useMemo(
    () => data.filter((row) =>
      columns.some((c) => String(row[c.key]).toLowerCase().includes(filter.toLowerCase()))
    ),
    [data, filter, columns]
  );

  const sorted = useMemo(() => {
    if (!sortKey) return filtered;
    return [...filtered].sort((a, b) => {
      const cmp = String(a[sortKey]).localeCompare(String(b[sortKey]));
      return sortDir === "asc" ? cmp : -cmp;
    });
  }, [filtered, sortKey, sortDir]);

  const toggleSort = (key: keyof T) => {
    if (sortKey === key) setSortDir((d) => (d === "asc" ? "desc" : "asc"));
    else { setSortKey(key); setSortDir("asc"); }
  };

  return (
    <>
      <input placeholder="Filter..." value={filter} onChange={(e) => setFilter(e.target.value)} />
      <table>
        <thead>
          <tr>
            {columns.map((c) => (
              <th key={String(c.key)} onClick={() => c.sortable && toggleSort(c.key)}>
                {c.label} {sortKey === c.key ? (sortDir === "asc" ? "↑" : "↓") : ""}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {sorted.map((row, i) => (
            <tr key={i}>
              {columns.map((c) => (
                <td key={String(c.key)}>{String(row[c.key])}</td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </>
  );
}
```

---

## 6. Multi-Select Tags Input

```tsx
function TagsInput({ tags, onChange }: { tags: string[]; onChange: (t: string[]) => void }) {
  const [input, setInput] = useState("");

  const addTag = () => {
    const t = input.trim();
    if (t && !tags.includes(t)) onChange([...tags, t]);
    setInput("");
  };

  return (
    <div className="tags-input">
      {tags.map((tag) => (
        <span key={tag} className="tag">
          {tag}
          <button aria-label={`Remove ${tag}`} onClick={() => onChange(tags.filter((x) => x !== tag))}>×</button>
        </span>
      ))}
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyDown={(e) => {
          if (e.key === "Enter") { e.preventDefault(); addTag(); }
          if (e.key === "Backspace" && !input && tags.length) onChange(tags.slice(0, -1));
        }}
      />
    </div>
  );
}
```

---

## 7. Star Rating

```tsx
function StarRating({ value, onChange, max = 5 }: { value: number; onChange?: (v: number) => void; max?: number }) {
  const [hover, setHover] = useState(0);
  const display = hover || value;

  return (
    <div role={onChange ? "slider" : "img"} aria-label={`${value} of ${max} stars`}>
      {Array.from({ length: max }, (_, i) => (
        <button
          key={i}
          type="button"
          disabled={!onChange}
          aria-label={`${i + 1} star`}
          onMouseEnter={() => onChange && setHover(i + 1)}
          onMouseLeave={() => setHover(0)}
          onClick={() => onChange?.(i + 1)}
          style={{ color: i < display ? "gold" : "#ccc" }}
        >
          ★
        </button>
      ))}
    </div>
  );
}
```

---

## 8. File Upload

```tsx
function FileUpload({ onFiles }: { onFiles: (files: File[]) => void }) {
  const [dragging, setDragging] = useState(false);
  const inputRef = useRef<HTMLInputElement>(null);

  const handleFiles = (list: FileList | null) => {
    if (!list?.length) return;
    onFiles(Array.from(list));
  };

  return (
    <div
      className={dragging ? "drop-zone active" : "drop-zone"}
      onDragOver={(e) => { e.preventDefault(); setDragging(true); }}
      onDragLeave={() => setDragging(false)}
      onDrop={(e) => { e.preventDefault(); setDragging(false); handleFiles(e.dataTransfer.files); }}
      onClick={() => inputRef.current?.click()}
    >
      <input ref={inputRef} type="file" multiple hidden onChange={(e) => handleFiles(e.target.files)} />
      <p>Drag & drop or click to upload</p>
    </div>
  );
}
```

---

## 9. Timer / Stopwatch

```tsx
function Stopwatch() {
  const [ms, setMs] = useState(0);
  const [running, setRunning] = useState(false);
  const startRef = useRef(0);
  const rafRef = useRef<number>();

  useEffect(() => {
    if (!running) return;
    startRef.current = performance.now() - ms;
    const tick = () => {
      setMs(performance.now() - startRef.current);
      rafRef.current = requestAnimationFrame(tick);
    };
    rafRef.current = requestAnimationFrame(tick);
    return () => cancelAnimationFrame(rafRef.current!);
  }, [running]);

  const format = (n: number) => {
    const s = Math.floor(n / 1000);
    const m = Math.floor(s / 60);
    return `${m}:${String(s % 60).padStart(2, "0")}.${Math.floor((n % 1000) / 10)}`;
  };

  return (
    <div>
      <span>{format(ms)}</span>
      <button onClick={() => setRunning(true)}>Start</button>
      <button onClick={() => setRunning(false)}>Pause</button>
      <button onClick={() => { setRunning(false); setMs(0); }}>Reset</button>
    </div>
  );
}
```

---

## 10. Interview Checklist

For any machine coding component:

- [ ] Controlled state + clear data flow
- [ ] Loading / error / empty states
- [ ] Keyboard support (Modal, Dropdown)
- [ ] Accessibility (roles, labels, focus)
- [ ] Cleanup (listeners, timers, abort)
- [ ] TypeScript props interface
- [ ] Mention how you'd test with RTL

---

*End of Machine Coding Classics Supplement*
