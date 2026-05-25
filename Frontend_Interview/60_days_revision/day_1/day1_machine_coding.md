# Day 1 — Machine Coding Revision

**React:** `useStack` (undo/redo) · multiple hook instances · undoable textarea  
**React Native:** Stack Navigator · navigation packages · passing params

---

## Table of Contents

### React (Web)
1. [Problem — useStack Hook](#1-problem--usestack-hook)
2. [useStack Implementation](#2-usestack-implementation)
3. [Two Instances of useStack in One File](#3-two-instances-of-usestack-in-one-file)
4. [Undoable Textarea — Full Solution](#4-undoable-textarea--full-solution)
5. [React Interview Points](#5-react-interview-points)

### React Native
6. [RN Project Setup](#6-rn-project-setup)
7. [Stack Navigator — Two Screens](#7-stack-navigator--two-screens)
8. [Pass Params Between Screens](#8-pass-params-between-screens)
9. [Navigation Packages — Features & Use Cases](#9-navigation-packages--features--use-cases)
10. [Navigation Interview Questions](#10-navigation-interview-questions)

---

# Part A — React (Web)

## 1. Problem — useStack Hook

**Task:** Build a custom hook `useStack` that supports **undo / redo** (like a text editor history).

**Requirements (typical interview):**

| Feature | Behavior |
|---------|----------|
| `push(value)` | Add new state to history |
| `undo()` | Go to previous state |
| `redo()` | Go to next state (after undo) |
| `canUndo` / `canRedo` | Disable buttons when not possible |
| Current value | What UI should display |

**Data structure (mental model):**

```
past:    [state0, state1, state2]   ← undo stack
present: state3                      ← current
future:  [state4]                    ← redo stack (cleared on new push)
```

When user types something new after undo → **clear `future`**.

---

## 2. useStack Implementation

```tsx
import { useCallback, useState } from "react";

type UseStackReturn<T> = {
  value: T;
  set: (next: T) => void;       // push new state (clears redo)
  undo: () => void;
  redo: () => void;
  canUndo: boolean;
  canRedo: boolean;
  reset: (initial: T) => void;
};

export function useStack<T>(initialValue: T): UseStackReturn<T> {
  const [past, setPast] = useState<T[]>([]);
  const [present, setPresent] = useState<T>(initialValue);
  const [future, setFuture] = useState<T[]>([]);

  const set = useCallback((next: T) => {
    setPast((p) => [...p, present]);
    setPresent(next);
    setFuture([]); // new edit clears redo branch
  }, [present]);

  const undo = useCallback(() => {
    if (past.length === 0) return;
    const previous = past[past.length - 1];
    setPast((p) => p.slice(0, -1));
    setFuture((f) => [present, ...f]);
    setPresent(previous);
  }, [past, present]);

  const redo = useCallback(() => {
    if (future.length === 0) return;
    const next = future[0];
    setFuture((f) => f.slice(1));
    setPast((p) => [...p, present]);
    setPresent(next);
  }, [future, present]);

  const reset = useCallback((initial: T) => {
    setPast([]);
    setPresent(initial);
    setFuture([]);
  }, []);

  return {
    value: present,
    set,
    undo,
    redo,
    canUndo: past.length > 0,
    canRedo: future.length > 0,
    reset,
  };
}
```

### Why this works

Each call to `useStack(initial)` creates **its own** `past`, `present`, `future` state inside that hook instance — React hooks isolate state per call site.

---

## 3. Two Instances of useStack in One File

### Rule

> **Every `useStack()` call = separate independent history.**

React does not share state between hook calls. Same as calling `useState()` twice gives two different counters.

```tsx
function App() {
  const titleStack = useStack("");      // instance 1
  const bodyStack = useStack("");       // instance 2 — completely separate

  return (
    <div>
      <input
        value={titleStack.value}
        onChange={(e) => titleStack.set(e.target.value)}
      />
      <button onClick={titleStack.undo} disabled={!titleStack.canUndo}>
        Undo Title
      </button>

      <textarea
        value={bodyStack.value}
        onChange={(e) => bodyStack.set(e.target.value)}
      />
      <button onClick={bodyStack.undo} disabled={!bodyStack.canUndo}>
        Undo Body
      </button>
    </div>
  );
}
```

### Interview answer (one line)

> Call `useStack` multiple times at the top level of the same component (or in different components). Each invocation gets its own `useState` closures — no shared `past`/`present`/`future`.

### What NOT to do

```tsx
// BAD — one shared stack used for two fields manually
const stack = useStack({ title: "", body: "" });
// Undo would revert BOTH together — usually not what you want
```

Use **one hook per independent undo history**.

### Optional: factory for named stacks

```tsx
function useNamedStacks<T extends Record<string, string>>(initial: T) {
  const keys = Object.keys(initial) as (keyof T)[];
  const stacks = Object.fromEntries(
    keys.map((key) => [key, useStack(initial[key])])
  ) as { [K in keyof T]: ReturnType<typeof useStack<string>> };
  return stacks;
}
// Prefer two explicit useStack() calls in interviews — clearer
```

---

## 4. Undoable Textarea — Full Solution

```tsx
import { useStack } from "./useStack";

export default function UndoableTextarea() {
  const { value, set, undo, redo, canUndo, canRedo } = useStack("");

  return (
    <div style={{ padding: 16, maxWidth: 480 }}>
      <h3>Undoable Textarea</h3>

      <textarea
        rows={6}
        style={{ width: "100%", padding: 8 }}
        value={value}
        onChange={(e) => set(e.target.value)}
        placeholder="Type here…"
      />

      <div style={{ display: "flex", gap: 8, marginTop: 12 }}>
        <button onClick={undo} disabled={!canUndo}>
          Undo
        </button>
        <button onClick={redo} disabled={!canRedo}>
          Redo
        </button>
      </div>

      <p style={{ fontSize: 12, color: "#666" }}>
        Undo: {canUndo ? "yes" : "no"} · Redo: {canRedo ? "yes" : "no"}
      </p>
    </div>
  );
}
```

### Debounced push (advanced — optional mention)

Pushing on every keystroke creates huge history. Interview bonus:

```tsx
// Push to stack only on blur or after 500ms debounce — not every onChange
```

### File structure (machine coding)

```
src/
  hooks/
    useStack.ts
  components/
    UndoableTextarea.tsx
  App.tsx
```

---

## 5. React Interview Points

| Question | Answer |
|----------|--------|
| How does undo/redo work? | Two stacks: `past` + `future`, `present` in middle |
| Why clear `future` on new `set`? | Standard editor behavior after new edit |
| Two `useStack` in one file? | Each call = own state (separate hook instances) |
| Why custom hook? | Reusable logic, testable, keeps UI thin |
| `useCallback` deps? | Include `present`, `past`, `future` used in closure |

---

# Part B — React Native

## 6. RN Project Setup

### Create project (Expo — fastest for interviews)

```bash
npx create-expo-app@latest MyApp
cd MyApp
npx expo install @react-navigation/native @react-navigation/native-stack
npx expo install react-native-screens react-native-safe-area-context
```

### Create project (CLI — bare workflow)

```bash
npx @react-native-community/cli init MyApp
cd MyApp
npm install @react-navigation/native @react-navigation/native-stack
npm install react-native-screens react-native-safe-area-context
# iOS only:
cd ios && pod install && cd ..
```

### Minimum files after setup

```
MyApp/
  App.tsx                 ← NavigationContainer + Stack
  screens/
    HomeScreen.tsx
    DetailsScreen.tsx
  package.json
```

---

## 7. Stack Navigator — Two Screens

```tsx
// App.tsx
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import HomeScreen from "./screens/HomeScreen";
import DetailsScreen from "./screens/DetailsScreen";

export type RootStackParamList = {
  Home: undefined;
  Details: { userId: string; userName: string };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

```tsx
// screens/HomeScreen.tsx
import { NativeStackScreenProps } from "@react-navigation/native-stack";
import { Button, Text, View } from "react-native";
import type { RootStackParamList } from "../App";

type Props = NativeStackScreenProps<RootStackParamList, "Home">;

export default function HomeScreen({ navigation }: Props) {
  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Text>Home Screen</Text>
      <Button
        title="Go to Details"
        onPress={() =>
          navigation.navigate("Details", {
            userId: "42",
            userName: "Chinmay",
          })
        }
      />
    </View>
  );
}
```

---

## 8. Pass Params Between Screens

### Send params (navigate)

```tsx
navigation.navigate("Details", {
  userId: "42",
  userName: "Chinmay",
});
```

### Receive params (destination screen)

```tsx
// screens/DetailsScreen.tsx
type Props = NativeStackScreenProps<RootStackParamList, "Details">;

export default function DetailsScreen({ route, navigation }: Props) {
  const { userId, userName } = route.params;

  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Text>User ID: {userId}</Text>
      <Text>Name: {userName}</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
    </View>
  );
}
```

### Type-safe params (must mention in interview)

```tsx
export type RootStackParamList = {
  Home: undefined;                    // no params
  Details: { userId: string; userName: string };
};
```

### Other navigation methods

| Method | Use case |
|--------|----------|
| `navigation.navigate("Screen", params)` | Go to screen; stays in stack |
| `navigation.push("Screen", params)` | Push another instance (stack only) |
| `navigation.replace("Screen", params)` | Replace current screen |
| `navigation.goBack()` | Pop current screen |
| `navigation.popToTop()` | Back to first screen in stack |
| `navigation.setParams({ ... })` | Update params of **current** screen |
| `navigation.reset({ ... })` | Reset entire navigation state |

### Pass params back to previous screen (callback pattern)

```tsx
// Home — pass callback via params (simple interviews)
navigation.navigate("Details", {
  userId: "1",
  onGoBack: (data: string) => setMessage(data),
});

// Details — call before goBack
route.params.onGoBack?.("updated from Details");
navigation.goBack();
```

> Production apps often prefer **global state**, **Context**, or **event emitters** instead of callback params.

---

## 9. Navigation Packages — Features & Use Cases

### Core packages (React Navigation v6+)

| Package | Role |
|---------|------|
| `@react-navigation/native` | Core library — `NavigationContainer`, hooks, types |
| `react-native-screens` | Native screen containers (performance) |
| `react-native-safe-area-context` | Safe area insets (notch, status bar) |

### Navigators (install per pattern)

| Package | Navigator | Use case |
|---------|-----------|----------|
| `@react-navigation/native-stack` | **Native Stack** | Most apps — iOS/Android native transitions, headers |
| `@react-navigation/stack` | **JS Stack** | Custom card animations, older apps |
| `@react-navigation/bottom-tabs` | **Bottom Tabs** | Main app sections (Home, Search, Profile) |
| `@react-navigation/drawer` | **Drawer** | Side menu navigation |
| `@react-navigation/material-top-tabs` | **Top Tabs** | Swipeable tabs under header |
| `@react-navigation/material-bottom-tabs` | **Material Bottom** | Material Design bottom bar |

### Feature comparison (interview table)

| Navigator | Animation | Performance | Typical use |
|-----------|-----------|-------------|-------------|
| **Native Stack** | Native platform | Best | Login → Home → Detail flows |
| **JS Stack** | JS-driven | Good | Heavy custom transitions |
| **Bottom Tabs** | Tab switch | Good | 3–5 main sections |
| **Drawer** | Slide menu | Good | Settings, many sections |
| **Top Tabs** | Horizontal swipe | Good | Categories, feeds |

### Supporting libraries

| Package | Purpose |
|---------|---------|
| `react-native-gesture-handler` | Gestures for drawer, stack swipe-back |
| `react-native-reanimated` | Smooth animations (drawer, tabs) |
| `@react-navigation/elements` | Shared header UI helpers |
| `expo-router` (Expo) | File-based routing on top of React Navigation |

### When to use what

```
Auth flow (Login, Signup)     → Native Stack
Main app (5 tabs)             → Bottom Tabs + nested Stacks per tab
Settings side menu            → Drawer
Product list → detail         → Stack inside a tab
Onboarding slides             → Stack or dedicated pager
Deep linking / web URLs       → NavigationContainer linking config
```

### Expo Router vs React Navigation

| | React Navigation | Expo Router |
|--|------------------|-------------|
| Routing | Code-based (`Stack.Screen`) | File-based (`app/home.tsx`) |
| Best for | Custom control, any RN project | Expo projects, web + mobile |
| Under the hood | Standalone | Built on React Navigation |

---

## 10. Navigation Interview Questions

| Question | Short answer |
|----------|--------------|
| How to pass data between screens? | `navigation.navigate("Screen", { params })` → `route.params` |
| Type-safe navigation? | `RootStackParamList` + `NativeStackScreenProps` |
| Difference `navigate` vs `push`? | `navigate` reuses existing screen if on stack; `push` always adds |
| Native Stack vs JS Stack? | Native = platform animations; JS = customizable card stack |
| What wraps the app? | `NavigationContainer` |
| Nested navigation? | Tab navigator → each tab has its own Stack |
| Deep linking? | `linking` prop on `NavigationContainer` |
| Hardware back (Android)? | Handled by stack; `beforeRemove` listener to confirm exit |
| Required peer deps? | `screens`, `safe-area-context`, often `gesture-handler` |

### Common machine coding checklist (RN)

- [ ] `NavigationContainer` at root  
- [ ] Typed `ParamList`  
- [ ] Two screens in Stack  
- [ ] Navigate with params  
- [ ] Display params on second screen  
- [ ] `goBack()` works  
- [ ] Mention package names and why Native Stack  

---

## Quick Revision — Day 1 Machine Coding

```
React:
  useStack → past + present + future
  2 instances → call useStack() twice = 2 separate states
  Textarea → value from hook, onChange → set(), buttons → undo/redo

React Native:
  @react-navigation/native + native-stack
  NavigationContainer → Stack.Navigator → Screens
  navigate("Details", { id }) → route.params.id
  Packages: native-stack (flows), bottom-tabs (main), drawer (menu)
```

---

*End of Day 1 machine coding*
