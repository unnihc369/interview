# Day 16 — LLD: State & Memento Patterns

**Topics:** State Pattern · Memento Pattern · Practical Examples · Interview Questions

---

# 1. State Pattern

## What is State Pattern?

Allows an object to alter its behavior when its internal state changes. Object appears to change its class.

---

# Problem Without State

```java
class Document {
    void publish() {
        if (status.equals("DRAFT")) { status = "REVIEW"; }
        else if (status.equals("REVIEW")) { status = "PUBLISHED"; }
        // growing if-else chain
    }
}
```

Violates Open/Closed Principle.

---

# Solution — State Pattern

```text
Context (Document)
    ↓ delegates to
State Interface
    ├── DraftState
    ├── ReviewState
    └── PublishedState
```

---

# State Pattern Implementation

```java
// State interface
interface DocumentState {
    void publish(Document doc);
    void reject(Document doc);
}

// Concrete states
class DraftState implements DocumentState {
    public void publish(Document doc) {
        System.out.println("Moving to review");
        doc.setState(new ReviewState());
    }
    public void reject(Document doc) {
        System.out.println("Already in draft");
    }
}

class ReviewState implements DocumentState {
    public void publish(Document doc) {
        System.out.println("Published!");
        doc.setState(new PublishedState());
    }
    public void reject(Document doc) {
        System.out.println("Back to draft");
        doc.setState(new DraftState());
    }
}

class PublishedState implements DocumentState {
    public void publish(Document doc) {
        System.out.println("Already published");
    }
    public void reject(Document doc) {
        throw new IllegalStateException("Cannot reject published doc");
    }
}

// Context
class Document {
    private DocumentState state = new DraftState();

    void setState(DocumentState state) { this.state = state; }
    void publish() { state.publish(this); }
    void reject() { state.reject(this); }
}
```

---

# Real-World Example — Vending Machine States

| State | Events |
|-------|--------|
| Idle | insertCoin → HasMoney |
| HasMoney | selectProduct → Dispensing |
| Dispensing | dispense → Idle |
| OutOfStock | any → reject |

Each state class handles transitions — no central switch statement.

---

# State vs Strategy (Interview)

| State | Strategy |
|-------|----------|
| Object changes behavior based on internal state | Client chooses algorithm externally |
| States know about transitions | Strategies are independent |
| Context delegates to current state | Context delegates to injected strategy |

---

# 2. Memento Pattern

## What is Memento Pattern?

Captures and restores object's internal state without exposing implementation details. Enables undo/rollback.

---

# Memento Structure

```text
Originator  → creates → Memento (snapshot)
Caretaker   → stores  → Memento
Originator  ← restores from Memento
```

---

# Memento Implementation

```java
// Memento — immutable snapshot
class EditorMemento {
    private final String content;

    EditorMemento(String content) {
        this.content = content;
    }

    String getContent() { return content; }
}

// Originator
class Editor {
    private String content = "";

    void write(String text) { content += text; }
    String getContent() { return content; }

    EditorMemento save() {
        return new EditorMemento(content);
    }

    void restore(EditorMemento memento) {
        this.content = memento.getContent();
    }
}

// Caretaker
class History {
    private final Deque<EditorMemento> stack = new ArrayDeque<>();

    void push(EditorMemento m) { stack.push(m); }

    EditorMemento pop() {
        return stack.isEmpty() ? null : stack.pop();
    }
}
```

---

# Usage — Undo Flow

```java
Editor editor = new Editor();
History history = new History();

editor.write("Hello ");
history.push(editor.save());

editor.write("World");
System.out.println(editor.getContent()); // Hello World

editor.restore(history.pop());
System.out.println(editor.getContent()); // Hello 
```

---

# Real-World Examples

| Domain | Memento Use |
|--------|-------------|
| Text editor | Undo/redo stack |
| Game | Save/load checkpoints |
| Database | Transaction rollback snapshots |
| Workflow | Restore previous approval state |

---

# 3. Combined Example — Document Editor with Undo

```java
class DocumentEditor {
    private DocumentState state = new DraftState();
    private final History history = new History();
    private String content = "";

    void type(String text) {
        history.push(new EditorMemento(content));
        content += text;
    }

    void undo() {
        EditorMemento m = history.pop();
        if (m != null) content = m.getContent();
    }

    void publish() { state.publish(wrapDocument()); }

    private Document wrapDocument() {
        Document doc = new Document();
        doc.setState(state);
        return doc;
    }
}
```

State handles workflow transitions. Memento handles content undo.

---

# 4. ATM Example Preview (Day 17)

State pattern maps naturally to ATM:

```text
IdleState → CardInsertedState → AuthenticatedState → TransactionState
```

Each state validates allowed operations.

---

# Interview Questions

## When to use State over if-else?

When behavior varies significantly across states and transitions are well-defined. Rule of thumb: 3+ states with different behavior per state.

---

## Memento vs deep clone?

Memento captures specific snapshot at point in time with controlled restore. Deep clone duplicates entire object graph — heavier and exposes internals.

---

## Who should create Memento — Originator or Caretaker?

**Originator** creates memento (knows internal structure). **Caretaker** only stores/restores — never inspects memento contents (encapsulation).

---

## Memory concern with Memento?

Unbounded undo stack grows memory. Solution: limit stack size (last N states) or store diffs instead of full snapshots.

---

## State pattern in Spring?

Spring bean lifecycle states (singleton, prototype) use different strategies. `@Scope` effectively selects instantiation strategy — related but not classic State pattern.

---

# Quick Revision

```text
State Pattern   → behavior changes with internal state, eliminates if-else
Memento Pattern → save/restore snapshots, undo functionality
Originator      → creates memento
Caretaker       → stores memento stack
State vs Strategy → internal transitions vs external algorithm choice
```

---

*End of Day 16 State & Memento LLD*
