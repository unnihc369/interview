# Day 1 — LeetCode (Stacks & Queues)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 150 | [Evaluate Reverse Polish Notation](#1-evaluate-reverse-polish-notation-150) | Medium | Stack |
| 394 | [Decode String](#2-decode-string-394) | Medium | Stack |
| 32 | [Longest Valid Parentheses](#3-longest-valid-parentheses-32) | Hard | Stack |

---

## Table of Contents

1. [Evaluate Reverse Polish Notation (150)](#1-evaluate-reverse-polish-notation-150)
2. [Decode String (394)](#2-decode-string-394)
3. [Longest Valid Parentheses (32)](#3-longest-valid-parentheses-32)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Evaluate Reverse Polish Notation (150)

### Problem (short)

Evaluate expression in **Reverse Polish Notation** (postfix).  
Operands and operators are given as strings. Valid operators: `+`, `-`, `*`, `/`. Division truncates toward zero.

**Example**

```
Input:  ["2", "1", "+", "3", "*"]
Think:  (2 + 1) * 3 = 9

Input:  ["4", "13", "5", "/", "+"]
Think:  4 + (13 / 5) = 4 + 2 = 6
```

### Hint 1 — What is RPN?

Read **left to right**. When you see a **number** → push it. When you see an **operator** → pop **two** numbers, compute, push result.

```
"2" → stack [2]
"1" → stack [2, 1]
"+" → pop 1, 2 → push 3        → [3]
"3" → stack [3, 3]
"*" → pop 3, 3 → push 9          → [9]
```

### Hint 2 — Order of operands matters

For `-` and `/`, the **first popped** is the **right** operand, the **second popped** is the **left**.

```
stack: [10, 3]
operator: "-"
a = pop() → 3   (right)
b = pop() → 10  (left)
result = b - a = 7
```

Same for `/`: `b / a` (integer division toward zero).

### Hint 3 — Algorithm steps

1. Loop each token in `tokens`.
2. If token is operator: pop `a`, pop `b`, push `b op a`.
3. Else: push `Number(token)`.
4. Return `stack[0]` at end.

### Walkthrough — `["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]`

Work in small chunks (interview: explain postfix grouping):

```
10, 6, 9, 3, +, -11, *, /, *, 17, +, 5, +
        ↓
      9+3=12 → (10, 6, 12)
      12*-11=-132 → (10, -132)
      10/(-132)=0 → (0)
      ... continue until one value left → 22
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) stack |

### Java

```java
class Solution {
    public int evalRPN(String[] tokens) {
        Deque<Integer> stack = new ArrayDeque<>();

        for (String token : tokens) {
            if (isOperator(token)) {
                int a = stack.pop(); // right
                int b = stack.pop(); // left
                switch (token) {
                    case "+": stack.push(b + a); break;
                    case "-": stack.push(b - a); break;
                    case "*": stack.push(b * a); break;
                    case "/": stack.push(b / a); break; // truncates toward 0
                }
            } else {
                stack.push(Integer.parseInt(token));
            }
        }
        return stack.pop();
    }

    private boolean isOperator(String s) {
        return s.length() == 1 && "+-*/".contains(s);
    }
}
```

### JavaScript

```js
/**
 * @param {string[]} tokens
 * @return {number}
 */
function evalRPN(tokens) {
  const stack = [];
  const ops = {
    "+": (b, a) => b + a,
    "-": (b, a) => b - a,
    "*": (b, a) => b * a,
    "/": (b, a) => Math.trunc(b / a), // toward zero
  };

  for (const token of tokens) {
    if (ops[token]) {
      const a = stack.pop();
      const b = stack.pop();
      stack.push(ops[token](b, a));
    } else {
      stack.push(Number(token));
    }
  }
  return stack[0];
}
```

### Common mistakes

- Wrong pop order for `-` and `/`.
- Using `Math.floor` for negative division in JS — use `Math.trunc`.
- Forgetting operators are strings (`"+"` not `+`).

---

## 2. Decode String (394)

### Problem (short)

Encoded string: `k[encoded_string]` means `encoded_string` repeated `k` times. Brackets can nest.

**Examples**

```
"3[a]2[bc]"     → "aaabcbc"
"3[a2[c]]"      → "accaccacc"
"2[abc]3[cd]ef" → "abcabccdcdcdef"
```

### Hint 1 — What to remember when you see `[`?

When entering a bracket group, you need to **save**:
- string built so far
- repeat count parsed before `[`

Then start fresh for inside the brackets.

### Hint 2 — Two stacks (classic approach)

| Stack | Stores |
|-------|--------|
| `countStack` | repeat numbers (e.g. `3` in `3[a]`) |
| `stringStack` | strings built before each `[` |

### Hint 3 — Process each character

| Char | Action |
|------|--------|
| Digit | Build `k` (may be multi-digit: `12[a]`) |
| `[` | Push current `curr` and `k` to stacks; reset `curr = ""`, `k = 0` |
| `]` | Pop prev string + count → `curr = prev + curr repeated k times` |
| Letter | Append to `curr` |

### Walkthrough — `"3[a2[c]]"`

```
'3' → k = 3
'[' → push curr="", k=3; reset
'a' → curr = "a"
'2' → k = 2
'[' → push curr="a", k=2; reset
'c' → curr = "c"
']' → curr = "a" + "c".repeat(2) = "acc"
']' → curr = "" + "acc".repeat(3) = "accaccacc"
```

### Hint 4 — Multi-digit numbers

```js
if (ch >= "0" && ch <= "9") {
  k = k * 10 + Number(ch);  // "12" → 12, not 1 then 2
}
```

### Complexity

| Time | Space |
|------|-------|
| O(n × max k) output size | O(n) stacks |

### Java

```java
class Solution {
    public String decodeString(String s) {
        Deque<Integer> countStack = new ArrayDeque<>();
        Deque<StringBuilder> stringStack = new ArrayDeque<>();
        StringBuilder curr = new StringBuilder();
        int k = 0;

        for (char ch : s.toCharArray()) {
            if (Character.isDigit(ch)) {
                k = k * 10 + (ch - '0');
            } else if (ch == '[') {
                countStack.push(k);
                stringStack.push(curr);
                curr = new StringBuilder();
                k = 0;
            } else if (ch == ']') {
                StringBuilder prev = stringStack.pop();
                int count = countStack.pop();
                for (int i = 0; i < count; i++) {
                    prev.append(curr);
                }
                curr = prev;
            } else {
                curr.append(ch);
            }
        }
        return curr.toString();
    }
}
```

### JavaScript

```js
/**
 * @param {string} s
 * @return {string}
 */
function decodeString(s) {
  const countStack = [];
  const stringStack = [];
  let curr = "";
  let k = 0;

  for (const ch of s) {
    if (ch >= "0" && ch <= "9") {
      k = k * 10 + Number(ch);
    } else if (ch === "[") {
      countStack.push(k);
      stringStack.push(curr);
      curr = "";
      k = 0;
    } else if (ch === "]") {
      const prev = stringStack.pop();
      const count = countStack.pop();
      curr = prev + curr.repeat(count);
    } else {
      curr += ch;
    }
  }
  return curr;
}
```

### Common mistakes

- Resetting `k` only on `[` but not after using it on `]`.
- Using `k = Number(ch)` inside loop for `12` — need `k = k * 10 + digit`.
- Appending repeat in wrong order: should be `prev + curr.repeat(k)`.

---

## 3. Longest Valid Parentheses (32)

### Problem (short)

Given string `s` with `(` and `)`, find **length of longest contiguous valid parentheses substring**.

**Examples**

```
"(()"        → 2   // "()"
")()())"     → 4   // "()()"
""          → 0
```

### Hint 1 — Brute force idea (why stack is better)

Check every substring — O(n³). Stack gives O(n).

### Hint 2 — Stack of indices (key idea)

Store **indices** of `(` on stack, not characters.

- Scan left to right with index `i`.
- `(` → push index `i`.
- `)` → if stack empty, this `)` has no match (skip). Else pop matching `(`, then valid length ending at `i` is `i - stack.peek()` (or `i + 1` if stack empty after pop).

### Hint 3 — Why index stack?

After popping match for `)`, the **previous unmatched index** on stack is the boundary before current valid segment.

```
index:  0 1 2 3 4 5
s:      ( ) ( ) ( )
              ↑ valid segment (2,3) length = 3-1 = 2? 

For ")()())" → longest is "()()" length 4
```

**Rule after pop:**

```text
if stack not empty:
  length = i - stack.peek()
else:
  length = i + 1   // whole segment from start is valid
maxLen = max(maxLen, length)
```

### Walkthrough — `")()())"`

```
i=0 ')' stack empty → skip
i=1 '(' push 1 → stack [1]
i=2 ')' pop → stack [] → len = 2-0+1? use i+1 = 2+1=2 → max=2
i=3 '(' push 3 → [3]
i=4 ')' pop → [] → len 4-0+1=4? actually stack empty → i+1 = 5? 

Careful walk:
i=2: pop 1, stack empty → length = i+1 = 3? 

Standard formula when stack empty after pop: length = i - (-1) effectively i+1
When stack has peek: length = i - peek

i=2: matched (1,2), stack empty → len = 2-(-1) = 3? 

Let me redo ")()())" properly:

s = ) ( ) ( ) )
0    )  skip
1    (  push 1  stack [1]
2    )  pop 1, stack [] → len = 2 - (-1) = 3? That's wrong for ")()"

Actually for ")()":
i=0 ) skip
i=1 ( push 1
i=2 ) pop, empty → len = i+1 = 3? but answer should be 2

The formula when stack is empty after pop: length = i - (-1) is wrong.

Correct: when stack empty after pop, length = i - (-1) means from index 0? 

Standard algorithm:
- Push -1 as sentinel initially on stack
- When pop and stack becomes empty after pop, length = i - (-1) = i+1? 

With sentinel -1:
stack = [-1]
i=1 '(' push 1 → [-1,1]
i=2 ')' pop → [-1], len = 2 - (-1) = 3 still wrong

Let me use the classic sentinel approach:

Initialize stack with -1
For i, char:
  if '(': push i
  if ')': 
    pop
    if stack empty: push i (new boundary)
    else: max = max(max, i - stack.peek())

Walk ")()())":
stack [-1]
i=0 ')': pop -1, stack empty, push 0 → [-1]? pop makes empty, push 0 → [0]
Actually: pop removes -1, empty, push i=0 → [0]
i=1 '(': push 1 → [0,1]
i=2 ')': pop 1, peek 0, len=2-0=2, max=2
i=3 '(': push 3 → [0,3]
i=4 ')': pop 3, peek 0, len=4-0=4, max=4
i=5 ')': pop 0, empty, push 5 → [5]
Answer 4 ✓
```

### Hint 4 — Sentinel `-1` on stack (simpler)

Start with `stack = [-1]` as "boundary before valid region".

On `)`:
1. Pop (remove last unmatched `(` index or sentinel).
2. If stack empty → push `i` as new boundary (this `)` broke validity).
3. Else → `maxLen = max(maxLen, i - stack.peek())`.

### Alternative — Two pointers O(1) space (bonus)

Left-to-right: balance `open`/`close`. When `close < 0`, reset. When equal, update max.  
Repeat right-to-left for cases like `(()`.

Interview: stack solution is enough; mention two-pass as follow-up.

### Complexity (stack)

| Time | Space |
|------|-------|
| O(n) | O(n) |

### Java

```java
class Solution {
    public int longestValidParentheses(String s) {
        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(-1); // sentinel
        int maxLen = 0;

        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.isEmpty()) {
                    stack.push(i); // boundary after invalid ')'
                } else {
                    maxLen = Math.max(maxLen, i - stack.peek());
                }
            }
        }
        return maxLen;
    }
}
```

### JavaScript

```js
/**
 * @param {string} s
 * @return {number}
 */
function longestValidParentheses(s) {
  const stack = [-1]; // sentinel
  let maxLen = 0;

  for (let i = 0; i < s.length; i++) {
    if (s[i] === "(") {
      stack.push(i);
    } else {
      stack.pop();
      if (stack.length === 0) {
        stack.push(i);
      } else {
        maxLen = Math.max(maxLen, i - stack[stack.length - 1]);
      }
    }
  }
  return maxLen;
}
```

### Common mistakes

- Pushing `(` characters instead of **indices**.
- Forgetting sentinel `-1` (makes length math harder).
- Not resetting boundary when extra `)` appears.

---

## 4. Pattern Cheat Sheet

| Problem | Stack stores | Trigger |
|---------|--------------|---------|
| **150 RPN** | Numbers | Operator → pop 2, compute, push |
| **394 Decode** | Prev strings + counts | `[` save, `]` merge repeat |
| **32 Longest Valid** | Indices (+ sentinel -1) | `)` → pop, measure segment length |

### When you see…

| Signal | Think |
|--------|-------|
| Postfix / calculator | Operand stack |
| Nested brackets + repeat | Two stacks (string + count) |
| Valid parentheses length | Index stack or two-pass balance |

### Solve order (revision)

1. **150** — Draw stack after each token; watch pop order for `-` `/`.
2. **394** — On `[` push state; on `]` repeat inner into outer.
3. **32** — Stack of indices, start with `-1`; on `)` update `maxLen`.

---

*End of Day 1 LeetCode*
