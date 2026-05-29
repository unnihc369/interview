# Day 4 — Machine Coding Revision

**React:** Call stack visualizer · debugger simulation  
**React Native:** Stack screen transitions · custom card style

---

## Table of Contents

### React (Web)

1. [Problem — Call Stack Visualizer](#1-problem--call-stack-visualizer)
2. [Simulated Code & Stack Frames](#2-simulated-code--stack-frames)
3. [Step-by-Step Debugger Engine](#3-step-by-step-debugger-engine)
4. [Full React Solution](#4-full-react-solution)
5. [React Interview Points](#5-react-interview-points)

### React Native

6. [Problem — Custom Card Transitions](#6-problem--custom-card-transitions)
7. [Stack Navigator Setup](#7-stack-navigator-setup)
8. [Custom Card Style Interpolator](#8-custom-card-style-interpolator)
9. [Full RN Solution](#9-full-rn-solution)
10. [RN Interview Points](#10-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Call Stack Visualizer

**Task:** Build a **debugger simulation** that visualizes the JavaScript **call stack** as predefined functions execute step-by-step.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Predefined script | Sequence of function calls (e.g., `main → foo → bar`) |
| Step / Run | Execute one frame or auto-run with delay |
| Stack display | LIFO list of active frames |
| Console output | Log statements as they "execute" |
| Highlight | Current executing line/frame highlighted |

---

## 2. Simulated Code & Stack Frames

```ts
type StackFrame = {
  id: string;
  fnName: string;
  line: number;
  locals: Record<string, unknown>;
};

type DebugStep =
  | { type: "ENTER"; fnName: string; line: number; locals?: Record<string, unknown> }
  | { type: "LOG"; message: string; fnName: string }
  | { type: "EXIT"; fnName: string }
  | { type: "ASSIGN"; fnName: string; name: string; value: unknown };

// Simulated execution trace for: main() → calculate(5) → double(5)
const EXECUTION_TRACE: DebugStep[] = [
  { type: "ENTER", fnName: "main", line: 1 },
  { type: "LOG", fnName: "main", message: "Program started" },
  { type: "ENTER", fnName: "calculate", line: 5, locals: { n: 5 } },
  { type: "LOG", fnName: "calculate", message: "Calculating for n=5" },
  { type: "ENTER", fnName: "double", line: 10, locals: { x: 5 } },
  { type: "ASSIGN", fnName: "double", name: "result", value: 10 },
  { type: "LOG", fnName: "double", message: "result = 10" },
  { type: "EXIT", fnName: "double" },
  { type: "ASSIGN", fnName: "calculate", name: "output", value: 10 },
  { type: "EXIT", fnName: "calculate" },
  { type: "LOG", fnName: "main", message: "Done: 10" },
  { type: "EXIT", fnName: "main" },
];
```

### Pseudo-source for display

```js
// Line numbers shown in UI
function main() {           // 1
  console.log("start");     // 2
  const r = calculate(5);   // 3
  console.log("Done:", r);  // 4
}
function calculate(n) {     // 5
  console.log("n=", n);       // 6
  return double(n);         // 7
}
function double(x) {        // 10
  const result = x * 2;     // 11
  console.log(result);      // 12
  return result;            // 13
}
```

---

## 3. Step-by-Step Debugger Engine

```tsx
import { useReducer, useCallback, useEffect, useRef } from "react";

type DebuggerState = {
  stack: StackFrame[];
  console: string[];
  stepIndex: number;
  isRunning: boolean;
  isFinished: boolean;
};

type Action =
  | { type: "STEP" }
  | { type: "RUN" }
  | { type: "PAUSE" }
  | { type: "RESET" };

let frameIdCounter = 0;

function debuggerReducer(state: DebuggerState, action: Action): DebuggerState {
  switch (action.type) {
    case "RESET":
      return {
        stack: [],
        console: [],
        stepIndex: 0,
        isRunning: false,
        isFinished: false,
      };

    case "STEP": {
      if (state.isFinished || state.stepIndex >= EXECUTION_TRACE.length) {
        return { ...state, isFinished: true, isRunning: false };
      }

      const step = EXECUTION_TRACE[state.stepIndex];
      let { stack, console: logs } = state;

      switch (step.type) {
        case "ENTER":
          stack = [
            ...stack,
            {
              id: String(++frameIdCounter),
              fnName: step.fnName,
              line: step.line,
              locals: step.locals ?? {},
            },
          ];
          break;
        case "LOG":
          logs = [...logs, `[${step.fnName}] ${step.message}`];
          break;
        case "ASSIGN": {
          const top = stack[stack.length - 1];
          if (top && top.fnName === step.fnName) {
            stack = [
              ...stack.slice(0, -1),
              {
                ...top,
                locals: { ...top.locals, [step.name]: step.value },
              },
            ];
          }
          break;
        }
        case "EXIT":
          stack = stack.slice(0, -1);
          break;
      }

      const nextIndex = state.stepIndex + 1;
      return {
        stack,
        console: logs,
        stepIndex: nextIndex,
        isRunning: state.isRunning,
        isFinished: nextIndex >= EXECUTION_TRACE.length,
      };
    }

    case "RUN":
      return { ...state, isRunning: true };

    case "PAUSE":
      return { ...state, isRunning: false };

    default:
      return state;
  }
}
```

---

## 4. Full React Solution

```tsx
const INITIAL_STATE: DebuggerState = {
  stack: [],
  console: [],
  stepIndex: 0,
  isRunning: false,
  isFinished: false,
};

export default function CallStackVisualizer() {
  const [state, dispatch] = useReducer(debuggerReducer, INITIAL_STATE);
  const intervalRef = useRef<ReturnType<typeof setInterval>>();

  const step = useCallback(() => dispatch({ type: "STEP" }), []);
  const reset = useCallback(() => {
    frameIdCounter = 0;
    dispatch({ type: "RESET" });
  }, []);

  useEffect(() => {
    if (state.isRunning && !state.isFinished) {
      intervalRef.current = setInterval(() => {
        dispatch({ type: "STEP" });
      }, 800);
    }
    return () => clearInterval(intervalRef.current);
  }, [state.isRunning, state.isFinished]);

  useEffect(() => {
    if (state.isFinished) dispatch({ type: "PAUSE" });
  }, [state.isFinished]);

  const currentFrame = state.stack[state.stack.length - 1];

  return (
    <div className="debugger">
      <h2>Call Stack Visualizer</h2>

      <div className="toolbar">
        <button onClick={step} disabled={state.isFinished}>Step ▶</button>
        <button
          onClick={() => dispatch({ type: state.isRunning ? "PAUSE" : "RUN" })}
          disabled={state.isFinished}
        >
          {state.isRunning ? "Pause ⏸" : "Run ⏩"}
        </button>
        <button onClick={reset}>Reset ↺</button>
        <span>Step {state.stepIndex}/{EXECUTION_TRACE.length}</span>
      </div>

      <div className="panels">
        <section className="source-panel">
          <h3>Source</h3>
          <pre>{PSEUDO_SOURCE}</pre>
          {currentFrame && (
            <p className="highlight">
              ▶ Executing: {currentFrame.fnName}() line {currentFrame.line}
            </p>
          )}
        </section>

        <section className="stack-panel">
          <h3>Call Stack</h3>
          <div className="stack-container">
            {state.stack.length === 0 ? (
              <p className="empty">Stack empty</p>
            ) : (
              [...state.stack].reverse().map((frame, i) => (
                <div
                  key={frame.id}
                  className={`frame ${i === 0 ? "active" : ""}`}
                >
                  <strong>{frame.fnName}()</strong>
                  <span>line {frame.line}</span>
                  {Object.keys(frame.locals).length > 0 && (
                    <pre>{JSON.stringify(frame.locals, null, 2)}</pre>
                  )}
                </div>
              ))
            )}
          </div>
        </section>

        <section className="console-panel">
          <h3>Console</h3>
          <div className="console-output">
            {state.console.map((line, i) => (
              <div key={i}>{line}</div>
            ))}
          </div>
        </section>
      </div>
    </div>
  );
}

const PSEUDO_SOURCE = `function main() {
  console.log("Program started");
  const r = calculate(5);
  console.log("Done:", r);
}
function calculate(n) {
  console.log("Calculating for n=" + n);
  return double(n);
}
function double(x) {
  const result = x * 2;
  console.log(result);
  return result;
}`;
```

### CSS highlights

```css
.debugger { font-family: "Fira Code", monospace; max-width: 900px; margin: 0 auto; }
.panels { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
.stack-container { display: flex; flex-direction: column-reverse; gap: 4px; }
.frame { border: 1px solid #ccc; padding: 8px; border-radius: 4px; background: #f9f9f9; }
.frame.active { border-color: #007bff; background: #e7f1ff; }
.console-output { background: #1e1e1e; color: #d4d4d4; padding: 12px; min-height: 120px; font-size: 13px; }
.highlight { color: #007bff; font-weight: bold; }
```

---

## 5. React Interview Points

| Question | Answer |
|----------|--------|
| Why useReducer? | Complex state: stack + console + step index |
| LIFO visualization | Reverse stack for display (top at top) |
| Real debugger? | Would parse AST / use VM; interview uses trace array |
| Auto-run cleanup | clearInterval on unmount and pause |
| Extend to breakpoints? | Pause when step.line matches breakpoint set |

### Checklist

- [ ] Stack grows on ENTER, shrinks on EXIT
- [ ] Step and Run modes
- [ ] Console log output
- [ ] Current frame highlighted
- [ ] Reset functionality

---

# Part B — React Native

## 6. Problem — Custom Card Transitions

**Task:** Configure React Navigation Stack with **custom card style transitions** (slide + fade + scale).

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Native Stack or JS Stack | Custom `cardStyleInterpolator` |
| Transition | Slide from right + opacity fade |
| Gesture | Swipe back enabled |
| Type-safe params | Typed `ParamList` |

---

## 7. Stack Navigator Setup

```tsx
// App.tsx
import { NavigationContainer } from "@react-navigation/native";
import { createStackNavigator, TransitionPresets } from "@react-navigation/stack";
import HomeScreen from "./screens/HomeScreen";
import DetailsScreen from "./screens/DetailsScreen";

export type RootStackParamList = {
  Home: undefined;
  Details: { itemId: string; title: string };
};

const Stack = createStackNavigator<RootStackParamList>();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerShown: true,
          gestureEnabled: true,
          cardStyleInterpolator: customCardStyleInterpolator,
          ...TransitionPresets.SlideFromRightIOS,
        }}
      >
        <Stack.Screen name="Home" component={HomeScreen} options={{ title: "Home" }} />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={{ title: "Details" }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

---

## 8. Custom Card Style Interpolator

```tsx
import {
  StackCardStyleInterpolator,
  StackCardInterpolationProps,
} from "@react-navigation/stack";

export const customCardStyleInterpolator: StackCardStyleInterpolator = ({
  current,
  next,
  layouts,
  inverted,
}: StackCardInterpolationProps) => {
  const progress = current.progress;

  // Slide from right
  const translateX = progress.interpolate({
    inputRange: [0, 1],
    outputRange: [layouts.screen.width, 0],
  });

  // Fade in
  const opacity = progress.interpolate({
    inputRange: [0, 0.5, 1],
    outputRange: [0, 0.5, 1],
  });

  // Slight scale for depth
  const scale = progress.interpolate({
    inputRange: [0, 1],
    outputRange: [0.95, 1],
  });

  // Next screen scales down slightly when current pushes
  const nextScale = next
    ? next.progress.interpolate({
        inputRange: [0, 1],
        outputRange: [1, 0.92],
      })
    : 1;

  return {
    cardStyle: {
      transform: [{ translateX }, { scale }],
      opacity,
    },
    overlayStyle: {
      opacity: progress.interpolate({
        inputRange: [0, 1],
        outputRange: [0, 0.3],
      }),
    },
  };
};
```

### Per-screen override

```tsx
<Stack.Screen
  name="Details"
  component={DetailsScreen}
  options={{
    cardStyleInterpolator: ({ current, layouts }) => ({
      cardStyle: {
        transform: [
          {
            translateY: current.progress.interpolate({
              inputRange: [0, 1],
              outputRange: [layouts.screen.height, 0],
            }),
          },
        ],
      },
    }),
  }}
/>
```

---

## 9. Full RN Solution

```tsx
// screens/HomeScreen.tsx
import { StackScreenProps } from "@react-navigation/stack";
import { View, Text, Button, StyleSheet } from "react-native";
import type { RootStackParamList } from "../App";

type Props = StackScreenProps<RootStackParamList, "Home">;

export default function HomeScreen({ navigation }: Props) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home Screen</Text>
      <Button
        title="Go to Details (Custom Transition)"
        onPress={() =>
          navigation.navigate("Details", {
            itemId: "42",
            title: "Custom Card Demo",
          })
        }
      />
    </View>
  );
}

// screens/DetailsScreen.tsx
type DetailProps = StackScreenProps<RootStackParamList, "Details">;

export default function DetailsScreen({ route, navigation }: DetailProps) {
  const { itemId, title } = route.params;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>{title}</Text>
      <Text>Item ID: {itemId}</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center", padding: 16 },
  title: { fontSize: 22, fontWeight: "bold", marginBottom: 16 },
});
```

### Package install

```bash
npm install @react-navigation/native @react-navigation/stack
npm install react-native-screens react-native-safe-area-context
npm install react-native-gesture-handler react-native-reanimated
```

---

## 10. RN Interview Points

| Question | Answer |
|----------|--------|
| Native Stack vs JS Stack for custom anim? | JS Stack (`@react-navigation/stack`) supports `cardStyleInterpolator` |
| Native Stack alternative | `animation`, `presentation` options; limited vs full custom |
| `cardStyleInterpolator` return shape | `{ cardStyle, overlayStyle, shadowStyle }` |
| Reanimated integration | `@react-navigation/stack` uses Animated; Reanimated 2+ for perf |
| Gesture back | `gestureEnabled: true` + gesture-handler |

### Checklist

- [ ] Stack Navigator with 2 screens
- [ ] Custom cardStyleInterpolator
- [ ] Typed ParamList
- [ ] Navigate with params
- [ ] Mention TransitionPresets as starting point

---

## Quick Revision — Day 4 Machine Coding

```
React:
  Debug trace array → ENTER/LOG/ASSIGN/EXIT
  useReducer → stack LIFO, console logs
  Step vs Run (setInterval)
  Display stack reversed (top first)

React Native:
  @react-navigation/stack (not native-stack for full custom)
  cardStyleInterpolator → translateX, opacity, scale
  TransitionPresets.SlideFromRightIOS as base
  gestureEnabled for swipe back
```

---

*End of Day 4 machine coding*
