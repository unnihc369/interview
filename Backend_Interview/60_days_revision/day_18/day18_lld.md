# Day 18 — LLD: Template Method & Visitor Patterns

**Topics:** Template Method Pattern · Visitor Pattern · Comparison · Interview Examples

---

# 1. Template Method Pattern

## What is Template Method?

Defines skeleton of algorithm in base class. Subclasses override specific steps without changing algorithm structure.

---

# Problem Without Template Method

```java
class PDFReport {
    void generate() {
        collectData();
        formatAsPDF();
        sendEmail();
    }
}

class HTMLReport {
    void generate() {
        collectData();
        formatAsHTML();
        sendEmail();
    }
}
```

Duplicate `collectData()` and `sendEmail()` logic.

---

# Solution

```java
abstract class ReportGenerator {

    // Template method — defines algorithm skeleton
    public final void generateReport() {
        collectData();
        formatReport();
        sendReport();
    }

    protected abstract void collectData();
    protected abstract void formatReport();

    protected void sendReport() {
        System.out.println("Sending report via email");
    }
}

class PDFReportGenerator extends ReportGenerator {
    protected void collectData() { /* query DB */ }
    protected void formatReport() { /* generate PDF */ }
}

class HTMLReportGenerator extends ReportGenerator {
    protected void collectData() { /* query DB */ }
    protected void formatReport() { /* generate HTML */ }
}
```

---

# Key Points

- `generateReport()` is `final` — subclasses cannot change algorithm order
- Hook methods can have default implementation
- Subclasses implement abstract steps

---

# Real-World Example — Data Processing Pipeline

```java
abstract class DataProcessor {

    public final void process() {
        validateInput();
        transform();
        persist();
        notify();
    }

    protected void validateInput() { /* common validation */ }
    protected abstract void transform();
    protected abstract void persist();
    protected void notify() { /* optional hook — override if needed */ }
}
```

Used in Spring: `JdbcTemplate`, `RestTemplate`, `JmsTemplate` — fixed workflow, customizable callbacks.

---

# 2. Visitor Pattern

## What is Visitor Pattern?

Separates algorithm from object structure. Add new operations without modifying existing classes.

---

# Problem

Object structure (AST, file system) needs multiple operations (export XML, export JSON, calculate size). Adding operation requires modifying every class.

---

# Solution Structure

```text
Element (interface)
  ├── accept(Visitor v)
  └── ConcreteElements

Visitor (interface)
  ├── visit(ConcreteElementA)
  └── visit(ConcreteElementB)
```

---

# Visitor Implementation

```java
// Element interface
interface Shape {
    void accept(ShapeVisitor visitor);
}

class Circle implements Shape {
    private double radius;
    public Circle(double radius) { this.radius = radius; }
    public double getRadius() { return radius; }
    public void accept(ShapeVisitor visitor) { visitor.visit(this); }
}

class Rectangle implements Shape {
    private double width, height;
    public Rectangle(double w, double h) { width = w; height = h; }
    public double getWidth() { return width; }
    public double getHeight() { return height; }
    public void accept(ShapeVisitor visitor) { visitor.visit(this); }
}

// Visitor interface
interface ShapeVisitor {
    void visit(Circle circle);
    void visit(Rectangle rectangle);
}

// Concrete visitor — Area calculation
class AreaCalculator implements ShapeVisitor {
    private double totalArea = 0;

    public void visit(Circle circle) {
        totalArea += Math.PI * circle.getRadius() * circle.getRadius();
    }

    public void visit(Rectangle rectangle) {
        totalArea += rectangle.getWidth() * rectangle.getHeight();
    }

    public double getTotalArea() { return totalArea; }
}

// Usage
List<Shape> shapes = List.of(new Circle(5), new Rectangle(3, 4));
AreaCalculator calculator = new AreaCalculator();
shapes.forEach(s -> s.accept(calculator));
System.out.println("Total area: " + calculator.getTotalArea());
```

---

# Add New Operation Without Modifying Shapes

```java
class PerimeterCalculator implements ShapeVisitor {
    private double total = 0;
    public void visit(Circle c) { total += 2 * Math.PI * c.getRadius(); }
    public void visit(Rectangle r) { total += 2 * (r.getWidth() + r.getHeight()); }
    public double getTotal() { return total; }
}
```

---

# 3. Template Method vs Visitor

| Feature | Template Method | Visitor |
|---------|-----------------|---------|
| Focus | Algorithm structure | Operations on object structure |
| Extension | Subclass overrides steps | New visitor class |
| Modifies | Base class hierarchy | Adds operations externally |
| Best for | Fixed workflow, variable steps | Stable structure, many operations |
| Example | Report generation | AST traversal, tax calculation |

---

# 4. Double Dispatch in Visitor

Why `accept(visitor)` on element?

1. First dispatch: `shape.accept(visitor)` → resolves to Circle.accept
2. Second dispatch: `visitor.visit(this)` → resolves to visit(Circle)

Enables type-specific behavior without instanceof chains.

---

# 5. Real-World Visitor — File System

```java
interface FileSystemNode {
    void accept(FSVisitor visitor);
}

class File implements FileSystemNode {
    private String name;
    private long size;
    public void accept(FSVisitor v) { v.visit(this); }
}

class Directory implements FileSystemNode {
    private List<FileSystemNode> children;
    public void accept(FSVisitor v) {
        v.visit(this);
        children.forEach(c -> c.accept(v));
    }
}

class SizeCalculator implements FSVisitor {
    private long totalSize = 0;
    public void visit(File f) { totalSize += f.getSize(); }
    public void visit(Directory d) { /* traverse children via accept */ }
}
```

---

# 6. Spring Framework Examples

| Pattern | Spring Example |
|---------|----------------|
| Template Method | `JdbcTemplate.query()` — fixed connection/statement flow |
| Template Method | `AbstractController` lifecycle methods |
| Visitor | Less common; compiler annotation processors use similar traversal |

---

# Interview Questions

## When to use Template Method?

When multiple classes share same algorithm structure but differ in specific steps. Example: payment processing (validate → charge → receipt) across PayPal, Stripe, Razorpay.

---

## Visitor downside?

Adding new element type requires updating ALL visitor interfaces and implementations. Structure changes are expensive.

**Rule:** Use Visitor when object structure is stable, operations change frequently.

---

## Template Method — can subclass skip a step?

Only if base class provides hook with empty default. `final` template method prevents skipping.

---

## Visitor vs Strategy?

Strategy: one algorithm, interchangeable. Visitor: many operations across heterogeneous object structure.

---

## Practical interview example for Template Method?

Design notification system: `sendNotification()` template with steps `validate()` → `formatMessage()` → `deliver()`. Subclasses implement Email, SMS, Push delivery.

---

# Quick Revision

```text
Template Method → skeleton in base class, steps in subclasses
Visitor         → separate operations from object structure
Double dispatch → accept() + visit() for type-specific behavior
Template Method → JdbcTemplate, report generators
Visitor         → AST, file system, tax on cart items
Visitor tradeoff → hard to add new element types
```

---

*End of Day 18 Template Method & Visitor LLD*
