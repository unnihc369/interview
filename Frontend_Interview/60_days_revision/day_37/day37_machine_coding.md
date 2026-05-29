# Day 37 — Machine Coding Revision

**React:** Tree editor add/remove nodes · BST phonebook UI  
**React Native:** Contact BST phonebook · sorted contacts

---

## Table of Contents

1. [Tree Editor — Add/Remove — React](#1-tree-editor--addremove--react)
2. [BST Phonebook Component](#2-bst-phonebook-component)
3. [Contact CRUD Operations](#3-contact-crud-operations)
4. [RN Sorted Contact List](#4-rn-sorted-contact-list)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Tree Editor — Add/Remove — React

```tsx
type Contact = { id: string; name: string; phone: string };

function contactsToBST(contacts: Contact[]): TreeNode | null {
  let root: TreeNode | null = null;
  for (const c of contacts.sort((a, b) => a.name.localeCompare(b.name))) {
    root = insertByKey(root, c.name, c);
  }
  return root;
}

function insertByKey(root: TreeNode | null, key: string, data: Contact): TreeNode {
  if (!root) return { val: key, data, left: null, right: null };
  if (key < root.val) root.left = insertByKey(root.left, key, data);
  else root.right = insertByKey(root.right, key, data);
  return root;
}
```

```tsx
function PhonebookEditor() {
  const [root, setRoot] = useState<TreeNode | null>(null);
  const [name, setName] = useState("");
  const [phone, setPhone] = useState("");

  const addContact = () => {
    const contact = { id: crypto.randomUUID(), name, phone };
    setRoot((r) => insertByKey(r, name, contact));
    setName(""); setPhone("");
  };

  const removeContact = (key: string) => {
    setRoot((r) => deleteByKey(r, key));
  };

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Name" />
      <input value={phone} onChange={(e) => setPhone(e.target.value)} placeholder="Phone" />
      <button onClick={addContact}>Add</button>
      <InorderList root={root} onDelete={removeContact} />
    </div>
  );
}

function InorderList({ root, onDelete }: Props) {
  const list = useMemo(() => inorderContacts(root), [root]);
  return (
    <ul>
      {list.map((c) => (
        <li key={c.id}>
          {c.name} — {c.phone}
          <button onClick={() => onDelete(c.name)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## 2. BST Phonebook Component

Display as sorted list (inorder) + optional tree visual side-by-side.

```ts
function inorderContacts(root: TreeNode | null, out: Contact[] = []): Contact[] {
  if (!root) return out;
  inorderContacts(root.left, out);
  out.push(root.data);
  inorderContacts(root.right, out);
  return out;
}
```

---

## 3. Contact CRUD Operations

| Op | BST op | UI |
|----|--------|-----|
| Create | Insert | Form submit |
| Read | Search / inorder | List view |
| Update | Delete + insert | Edit modal |
| Delete | deleteNode | Swipe / button |

**Update pattern:** Delete old key, insert new key (name change).

---

## 4. RN Sorted Contact List

```tsx
function ContactPhonebook() {
  const [root, setRoot] = useState<TreeNode | null>(null);
  const contacts = useMemo(() => inorderContacts(root), [root]);

  const renderItem = ({ item }: { item: Contact }) => (
    <Swipeable
      renderRightActions={() => (
        <Pressable onPress={() => setRoot((r) => deleteByKey(r, item.name))}>
          <Text style={{ color: "red", padding: 16 }}>Delete</Text>
        </Pressable>
      )}
    >
      <View style={{ padding: 12, borderBottomWidth: 1, borderColor: "#eee" }}>
        <Text style={{ fontWeight: "600" }}>{item.name}</Text>
        <Text>{item.phone}</Text>
      </View>
    </Swipeable>
  );

  return (
    <FlatList
      data={contacts}
      keyExtractor={(c) => c.id}
      renderItem={renderItem}
      ListHeaderComponent={<AddContactForm onAdd={(c) => setRoot((r) => insertByKey(r, c.name, c))} />}
    />
  );
}
```

Use `SectionList` with first-letter headers for UX (inorder gives alphabetical order).

---

## 5. Interview Checklist

- [ ] Insert maintains BST order by name
- [ ] Delete handles 0/1/2 children
- [ ] Inorder renders sorted list
- [ ] RN swipe-to-delete

---

*End of Day 37 machine coding*
