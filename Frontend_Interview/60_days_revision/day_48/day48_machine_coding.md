# Day 48 — Machine Coding Revision

**React:** IP routing table simulator  
**React Native:** Network prefix match

---

## Table of Contents

### React (Web)

1. [Problem — IP Route Lookup](#1-problem--ip-route-lookup)
2. [Binary Trie Router](#2-binary-trie-router)
3. [React IP Router UI](#3-react-ip-router-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Network Prefix Matcher](#5-network-prefix-matcher)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — IP Route Lookup

**Task:** Visual IP routing table. Add CIDR routes with labels. Lookup IP → **longest prefix match** result.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Add route | CIDR + label |
| Lookup | longest prefix wins |
| Validate | IPv4 format |
| Show path | bits walked in trie |

---

## 2. Binary Trie Router

```js
function ipToBits(ip) {
  const parts = ip.split(".").map(Number);
  if (parts.length !== 4 || parts.some((p) => p < 0 || p > 255 || isNaN(p))) return null;
  return parts.map((p) => p.toString(2).padStart(8, "0")).join("");
}

class IPTrieRouter {
  constructor() {
    this.root = { zero: null, one: null, label: null, prefixLen: 0 };
  }

  addRoute(cidr, label) {
    const [ip, lenStr] = cidr.split("/");
    const prefixLen = parseInt(lenStr, 10);
    const bits = ipToBits(ip);
    if (!bits || prefixLen < 0 || prefixLen > 32) return false;

    let node = this.root;
    for (let i = 0; i < prefixLen; i++) {
      const bit = bits[i];
      if (bit === "0") {
        if (!node.zero) node.zero = { zero: null, one: null, label: null, prefixLen: 0 };
        node = node.zero;
      } else {
        if (!node.one) node.one = { zero: null, one: null, label: null, prefixLen: 0 };
        node = node.one;
      }
    }
    node.label = label;
    node.prefixLen = prefixLen;
    return true;
  }

  lookup(ip) {
    const bits = ipToBits(ip);
    if (!bits) return { label: null, path: [] };

    let node = this.root;
    let best = { label: null, prefixLen: 0 };
    const path = [];

    for (let i = 0; i < 32 && node; i++) {
      if (node.label) best = { label: node.label, prefixLen: node.prefixLen };
      path.push(bits[i]);
      node = bits[i] === "0" ? node.zero : node.one;
    }
    if (node?.label && node.prefixLen >= best.prefixLen) {
      best = { label: node.label, prefixLen: node.prefixLen };
    }
    return { ...best, path: path.slice(0, best.prefixLen || path.length) };
  }
}

const router = new IPTrieRouter();
router.addRoute("192.168.0.0/16", "Private LAN");
router.addRoute("192.168.1.0/24", "Office WiFi");
router.addRoute("10.0.0.0/8", "VPN");
```

---

## 3. React IP Router UI

```tsx
import { useState, useMemo } from "react";

export default function IPRouterSimulator() {
  const [ip, setIp] = useState("192.168.1.100");
  const [cidr, setCidr] = useState("172.16.0.0/12");
  const [label, setLabel] = useState("Guest Network");
  const [routes, setRoutes] = useState([
    { cidr: "192.168.0.0/16", label: "Private LAN" },
    { cidr: "192.168.1.0/24", label: "Office WiFi" },
    { cidr: "10.0.0.0/8", label: "VPN" },
  ]);
  const [rt] = useState(() => new IPTrieRouter());

  useMemo(() => {
    routes.forEach((r) => rt.addRoute(r.cidr, r.label));
  }, [routes]);

  const result = useMemo(() => rt.lookup(ip), [ip, routes]);

  const addRoute = () => {
    if (rt.addRoute(cidr, label)) setRoutes((r) => [...r, { cidr, label }]);
  };

  return (
    <div style={{ maxWidth: 560, margin: 24, fontFamily: "monospace" }}>
      <h2 style={{ fontFamily: "system-ui" }}>IP Routing Table (Binary Trie)</h2>

      <div style={{ marginBottom: 16 }}>
        <label>Lookup IP: </label>
        <input value={ip} onChange={(e) => setIp(e.target.value)} style={{ padding: 8, width: 180 }} />
      </div>

      <div style={{
        padding: 16, background: result.label ? "#e8f5e9" : "#ffebee",
        borderRadius: 8, marginBottom: 16,
      }}>
        <div>Match: <strong>{result.label ?? "NO ROUTE"}</strong></div>
        {result.prefixLen > 0 && <div>Prefix length: /{result.prefixLen}</div>}
        <div style={{ fontSize: 11, marginTop: 8, wordBreak: "break-all" }}>
          Path: {result.path?.join("") || "—"}
        </div>
      </div>

      <fieldset style={{ border: "1px solid #ccc", padding: 12, borderRadius: 8 }}>
        <legend>Add Route</legend>
        <input value={cidr} onChange={(e) => setCidr(e.target.value)} placeholder="CIDR" style={{ marginRight: 8, padding: 6 }} />
        <input value={label} onChange={(e) => setLabel(e.target.value)} placeholder="Label" style={{ marginRight: 8, padding: 6 }} />
        <button onClick={addRoute}>Add</button>
      </fieldset>

      <table style={{ width: "100%", marginTop: 16, fontSize: 13 }}>
        <thead><tr><th>CIDR</th><th>Label</th></tr></thead>
        <tbody>
          {routes.map((r, i) => (
            <tr key={i}><td>{r.cidr}</td><td>{r.label}</td></tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Longest prefix match | Track best at every level during 32-bit walk |
| IPv6 | 128 levels — same binary trie |
| CIDR validation | Parse IP + mask length 0-32 |
| vs linear route table | Trie O(32) vs O(routes) scan |

---

# Part B — React Native

## 5. Network Prefix Matcher

**Task:** Check if device is on allowed network (corporate WiFi / VPN) using CIDR binary trie before sensitive API call.

```tsx
import { useState, useMemo } from "react";
import { View, Text, TextInput, Pressable, StyleSheet, Alert } from "react-native";

// Reuse IPTrieRouter from above
const ALLOWED_NETWORKS = [
  { cidr: "10.0.0.0/8", label: "Corporate VPN" },
  { cidr: "192.168.0.0/16", label: "Office WiFi" },
];

function buildAllowedRouter() {
  const rt = new IPTrieRouter();
  ALLOWED_NETWORKS.forEach((n) => rt.addRoute(n.cidr, n.label));
  return rt;
}

const allowedRouter = buildAllowedRouter();

export default function NetworkPrefixMatch() {
  const [deviceIp, setDeviceIp] = useState("10.0.0.55");
  const match = useMemo(() => allowedRouter.lookup(deviceIp), [deviceIp]);

  const syncSecureData = () => {
    if (!match.label) {
      Alert.alert("Blocked", "Device not on allowed network");
      return;
    }
    Alert.alert("Sync OK", `Allowed via ${match.label}`);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Network Prefix Match</Text>
      <Text style={styles.sub}>Binary trie CIDR check before sync</Text>

      <TextInput
        value={deviceIp}
        onChangeText={setDeviceIp}
        placeholder="Device IP"
        style={styles.input}
        keyboardType="numeric"
      />

      <View style={[styles.badge, match.label ? styles.ok : styles.block]}>
        <Text>{match.label ? `✓ ${match.label}` : "✗ Not allowed"}</Text>
      </View>

      <Pressable style={styles.btn} onPress={syncSecureData}>
        <Text style={styles.btnText}>Sync Secure Data</Text>
      </Pressable>

      <Text style={styles.hint}>
        Allowed: {ALLOWED_NETWORKS.map((n) => n.cidr).join(", ")}
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, paddingTop: 60 },
  title: { fontSize: 20, fontWeight: "700" },
  sub: { color: "#666", marginBottom: 16, marginTop: 4 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 10, padding: 14, fontSize: 16, fontFamily: "monospace" },
  badge: { marginTop: 12, padding: 12, borderRadius: 8, alignItems: "center" },
  ok: { backgroundColor: "#e8f5e9" },
  block: { backgroundColor: "#ffebee" },
  btn: { marginTop: 20, backgroundColor: "#1976d2", padding: 14, borderRadius: 10, alignItems: "center" },
  btnText: { color: "#fff", fontWeight: "600" },
  hint: { marginTop: 16, fontSize: 12, color: "#888" },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Get device IP RN | `@react-native-community/netinfo` + native module |
| Security | Client-side check is UX gate only — server validates too |
| Offline | Trie in memory; no network needed for CIDR match |
| expo-network | `Network.getIpAddressAsync()` on LAN |

---

## Quick Revision — Day 48

```
Web:  CIDR binary trie + longest prefix lookup UI
RN:   Allowed network gate before secure sync
LC:   421 max XOR, 1707 range AND, 1938 genetic score
```

---

*End of Day 48 machine coding*
