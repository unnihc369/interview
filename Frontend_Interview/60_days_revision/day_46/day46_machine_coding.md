# Day 46 — Machine Coding Revision

**React:** URL router simulation  
**React Native:** Deep link trie router

---

## Table of Contents

### React (Web)

1. [Problem — Client-Side Router](#1-problem--client-side-router)
2. [Route Trie Implementation](#2-route-trie-implementation)
3. [React Router UI](#3-react-router-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Deep Link Trie Router](#5-deep-link-trie-router)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Client-Side Router

**Task:** Build a minimal SPA router using a **trie of path segments**. Support static routes, `:params`, and 404 fallback.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Static routes | `/`, `/about` |
| Params | `/users/:id` |
| Nested | `/users/:id/posts` |
| 404 | Unknown paths |

---

## 2. Route Trie Implementation

```js
class RouteNode {
  constructor() {
    this.static = new Map();   // segment → RouteNode
    this.param = null;         // { name, child }
    this.component = null;
  }
}

class RouteTrie {
  constructor() {
    this.root = new RouteNode();
  }

  add(path, component) {
    const parts = path.split("/").filter(Boolean);
    let node = this.root;
    for (const part of parts) {
      if (part.startsWith(":")) {
        if (!node.param) node.param = { name: part.slice(1), child: new RouteNode() };
        node = node.param.child;
      } else {
        if (!node.static.has(part)) node.static.set(part, new RouteNode());
        node = node.static.get(part);
      }
    }
    node.component = component;
  }

  match(pathname) {
    const parts = pathname.split("/").filter(Boolean);
    const params = {};
    const result = this._match(this.root, parts, 0, params);
    return result ?? { component: "NotFound", params: {} };
  }

  _match(node, parts, i, params) {
    if (i === parts.length) {
      return node.component ? { component: node.component, params } : null;
    }
    const part = parts[i];
    if (node.static.has(part)) {
      const r = this._match(node.static.get(part), parts, i + 1, params);
      if (r) return r;
    }
    if (node.param) {
      params[node.param.name] = part;
      const r = this._match(node.param.child, parts, i + 1, params);
      if (r) return r;
      delete params[node.param.name];
    }
    return null;
  }
}

const routeTrie = new RouteTrie();
routeTrie.add("/", "Home");
routeTrie.add("/about", "About");
routeTrie.add("/users/:id", "UserProfile");
routeTrie.add("/users/:id/posts", "UserPosts");
```

---

## 3. React Router UI

```tsx
import { useState, useMemo } from "react";

const COMPONENTS: Record<string, React.FC<{ params: Record<string, string> }>> = {
  Home: () => <h2>Home Page</h2>,
  About: () => <h2>About Page</h2>,
  UserProfile: ({ params }) => <h2>User Profile: {params.id}</h2>,
  UserPosts: ({ params }) => <h2>Posts by User {params.id}</h2>,
  NotFound: () => <h2>404 — Not Found</h2>,
};

export default function TrieRouter() {
  const [path, setPath] = useState("/users/42/posts");

  const match = useMemo(() => routeTrie.match(path), [path]);
  const Page = COMPONENTS[match.component] ?? COMPONENTS.NotFound;

  const links = ["/", "/about", "/users/99", "/users/42/posts", "/unknown"];

  return (
    <div style={{ maxWidth: 560, margin: 24, fontFamily: "system-ui" }}>
      <h2>Trie URL Router</h2>
      <nav style={{ display: "flex", gap: 8, flexWrap: "wrap", marginBottom: 16 }}>
        {links.map((l) => (
          <button
            key={l}
            onClick={() => setPath(l)}
            style={{
              padding: "6px 12px",
              background: path === l ? "#1976d2" : "#eee",
              color: path === l ? "#fff" : "#333",
              border: "none", borderRadius: 6, cursor: "pointer",
            }}
          >
            {l}
          </button>
        ))}
      </nav>
      <div style={{ padding: 16, background: "#f5f5f5", borderRadius: 8 }}>
        <code>path: {path}</code>
        <pre style={{ fontSize: 12 }}>{JSON.stringify(match.params, null, 2)}</pre>
        <Page params={match.params} />
      </div>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Trie vs React Router | Real routers use similar segment trees |
| Static before param | `/users/new` vs `/users/:id` — static wins |
| History API | `pushState` + popstate listener |
| Code splitting | Lazy `import()` in route component field |

---

# Part B — React Native

## 5. Deep Link Trie Router

**Task:** Map deep link URLs to screens using trie. Handle `myapp://users/42/posts/7`.

```tsx
import { useState, useMemo } from "react";
import { View, Text, Pressable, StyleSheet, Linking } from "react-native";

type ScreenMatch = { screen: string; params: Record<string, string> };

class DeepLinkTrie {
  root = { static: new Map(), param: null as null | { name: string; child: any }, screen: null as string | null };

  add(pattern: string, screen: string) {
    const parts = pattern.split("/").filter(Boolean);
    let node = this.root;
    for (const part of parts) {
      if (part.startsWith(":")) {
        if (!node.param) node.param = { name: part.slice(1), child: { static: new Map(), param: null, screen: null } };
        node = node.param.child;
      } else {
        if (!node.static.has(part)) node.static.set(part, { static: new Map(), param: null, screen: null });
        node = node.static.get(part);
      }
    }
    node.screen = screen;
  }

  match(path: string): ScreenMatch | null {
    const parts = path.split("/").filter(Boolean);
    const params: Record<string, string> = {};
    const dfs = (node: any, i: number): ScreenMatch | null => {
      if (i === parts.length) return node.screen ? { screen: node.screen, params } : null;
      const part = parts[i];
      if (node.static.has(part)) {
        const r = dfs(node.static.get(part), i + 1);
        if (r) return r;
      }
      if (node.param) {
        params[node.param.name] = part;
        const r = dfs(node.param.child, i + 1);
        if (r) return r;
        delete params[node.param.name];
      }
      return null;
    };
    return dfs(this.root, 0);
  }
}

const deepLinkTrie = new DeepLinkTrie();
deepLinkTrie.add("users/:userId", "Profile");
deepLinkTrie.add("users/:userId/posts/:postId", "PostDetail");
deepLinkTrie.add("home", "Home");

function parseDeepLink(url: string) {
  const path = url.replace(/^(myapp:\/\/|https:\/\/app\.example\.com\/)/, "");
  return deepLinkTrie.match(path);
}

export default function DeepLinkRouter() {
  const [current, setCurrent] = useState<ScreenMatch | null>(null);

  const testLinks = [
    "myapp://home",
    "myapp://users/42",
    "myapp://users/42/posts/7",
    "myapp://unknown",
  ];

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Deep Link Router</Text>
      {testLinks.map((url) => (
        <Pressable key={url} style={styles.btn} onPress={() => setCurrent(parseDeepLink(url))}>
          <Text style={styles.url}>{url}</Text>
        </Pressable>
      ))}
      {current ? (
        <View style={styles.result}>
          <Text style={styles.screen}>Screen: {current.screen}</Text>
          <Text>{JSON.stringify(current.params)}</Text>
        </View>
      ) : (
        <Text style={styles.miss}>Tap a link (last unknown → null)</Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, paddingTop: 60 },
  title: { fontSize: 20, fontWeight: "700", marginBottom: 16 },
  btn: { padding: 12, backgroundColor: "#e3f2fd", borderRadius: 8, marginBottom: 8 },
  url: { fontSize: 13, fontFamily: "monospace" },
  result: { marginTop: 20, padding: 16, backgroundColor: "#f5f5f5", borderRadius: 8 },
  screen: { fontWeight: "700", fontSize: 16 },
  miss: { marginTop: 20, color: "#888" },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| React Navigation linking | Declarative config → internal trie |
| Universal links | iOS associated domains + same path trie |
| Cold start deep link | `Linking.getInitialURL()` on mount |
| Param validation | Validate `:id` is numeric in match handler |

---

## Quick Revision — Day 46

```
Web:  Segment trie router + static/param + 404
RN:   Deep link parse → trie match → navigate
LC:   1804 Trie II, 440 k-th lex, 386 lex numbers
```

---

*End of Day 46 machine coding*
