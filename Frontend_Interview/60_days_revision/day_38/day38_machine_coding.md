# Day 38 — Machine Coding Revision

**React:** Nested form validation tree · field-level errors  
**React Native:** Nested form validator · scroll to first error

---

## Table of Contents

1. [Nested Form Schema](#1-nested-form-schema)
2. [validateTree Function](#2-validatetree-function)
3. [FormTree Component — React](#3-formtree-component--react)
4. [RN Nested Validator](#4-rn-nested-validator)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Nested Form Schema

```ts
type FieldSchema = {
  id: string;
  label: string;
  type: "text" | "number" | "group";
  required?: boolean;
  min?: number;
  max?: number;
  children?: FieldSchema[];
};

type FormValues = Record<string, string | number>;

const schema: FieldSchema = {
  id: "user",
  label: "User",
  type: "group",
  children: [
    { id: "name", label: "Name", type: "text", required: true },
    {
      id: "address",
      label: "Address",
      type: "group",
      children: [
        { id: "city", label: "City", type: "text", required: true },
        { id: "zip", label: "ZIP", type: "text", required: true },
      ],
    },
  ],
};
```

---

## 2. validateTree Function

```ts
type FieldError = { id: string; message: string };

function validateField(node: FieldSchema, values: FormValues): FieldError[] {
  const errors: FieldError[] = [];

  if (node.type !== "group") {
    const val = values[node.id];
    if (node.required && (val === undefined || val === "")) {
      errors.push({ id: node.id, message: `${node.label} is required` });
    }
    if (node.type === "number" && typeof val === "number") {
      if (node.min !== undefined && val < node.min)
        errors.push({ id: node.id, message: `Min ${node.min}` });
      if (node.max !== undefined && val > node.max)
        errors.push({ id: node.id, message: `Max ${node.max}` });
    }
    return errors;
  }

  // Group: validate children only if group itself passes (optional group rules)
  for (const child of node.children ?? []) {
    errors.push(...validateField(child, values));
  }
  return errors;
}
```

**BST analogy:** Parent bounds must hold before validating subtree fields.

---

## 3. FormTree Component — React

```tsx
function FormField({ node, values, errors, onChange }: Props) {
  const error = errors.find((e) => e.id === node.id);

  if (node.type === "group") {
    return (
      <fieldset style={{ marginLeft: 16, border: "1px solid #ddd", padding: 12 }}>
        <legend>{node.label}</legend>
        {node.children?.map((c) => (
          <FormField key={c.id} node={c} values={values} errors={errors} onChange={onChange} />
        ))}
      </fieldset>
    );
  }

  return (
    <div style={{ marginBottom: 8 }}>
      <label>{node.label}</label>
      <input
        value={String(values[node.id] ?? "")}
        onChange={(e) => onChange(node.id, e.target.value)}
        style={{ borderColor: error ? "red" : "#ccc" }}
      />
      {error && <span style={{ color: "red", fontSize: 12 }}>{error.message}</span>}
    </div>
  );
}

export function NestedForm() {
  const [values, setValues] = useState<FormValues>({});
  const [errors, setErrors] = useState<FieldError[]>([]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const errs = validateField(schema, values);
    setErrors(errs);
    if (errs.length === 0) console.log("Valid!", values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <FormField node={schema} values={values} errors={errors} onChange={(id, v) => setValues((p) => ({ ...p, [id]: v }))} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## 4. RN Nested Validator

```tsx
function NestedFormRN() {
  const [values, setValues] = useState<FormValues>({});
  const [errors, setErrors] = useState<FieldError[]>([]);
  const scrollRef = useRef<ScrollView>(null);
  const fieldRefs = useRef<Map<string, number>>(new Map());

  const submit = () => {
    const errs = validateField(schema, values);
    setErrors(errs);
    if (errs.length) {
      const firstY = fieldRefs.current.get(errs[0].id) ?? 0;
      scrollRef.current?.scrollTo({ y: firstY, animated: true });
    }
  };

  return (
    <ScrollView ref={scrollRef}>
      <RecursiveFormFields schema={schema} values={values} errors={errors} fieldRefs={fieldRefs} onChange={setValues} />
      <Button title="Submit" onPress={submit} />
    </ScrollView>
  );
}
```

Use `onLayout` to record Y positions for scroll-to-error.

---

## 5. Interview Checklist

- [ ] Recursive schema validation
- [ ] Aggregate errors from subtree
- [ ] Highlight first invalid field
- [ ] RN scroll to error

---

*End of Day 38 machine coding*
