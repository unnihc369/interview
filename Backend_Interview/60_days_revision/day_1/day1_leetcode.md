# Day 1 — LeetCode (Arrays · Stack · Linked List)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 1 | [Two Sum](#1-two-sum-1) | Easy | Hash map |
| 20 | [Valid Parentheses](#2-valid-parentheses-20) | Easy | Stack |
| 21 | [Merge Two Sorted Lists](#3-merge-two-sorted-lists-21) | Easy | Linked list |

---

## Table of Contents

1. [Two Sum (1)](#1-two-sum-1)
2. [Valid Parentheses (20)](#2-valid-parentheses-20)
3. [Merge Two Sorted Lists (21)](#3-merge-two-sorted-lists-21)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Two Sum (1)

### Problem (short)

Given `nums` and `target`, return **indices** of two numbers that add up to `target`. Exactly one solution exists.

```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]    // nums[0] + nums[1] = 2 + 7 = 9
```

### Hint 1 — Brute force (baseline)

Two nested loops — O(n²). Mention it, then optimize.

### Hint 2 — One pass with hash map

For each `nums[i]`, you need `complement = target - nums[i]`.

If complement was seen before → return `[map.get(complement), i]`.

Else store `nums[i] → i` in map.

```
nums = [2, 7, 11, 15], target = 9

i=0, val=2, need 7 → not in map → map {2:0}
i=1, val=7, need 2 → in map at index 0 → return [0,1]
```

### Hint 3 — Why map stores value → index

You need **indices** in output, not values.

### Algorithm

1. `Map<Integer, Integer> map` (value → index).
2. For `i` from `0` to `n-1`:
   - `need = target - nums[i]`
   - If `map.containsKey(need)` → return `[map.get(need), i]`
   - `map.put(nums[i], i)`
3. Return `[]` (problem guarantees solution).

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### Java

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int need = target - nums[i];
            if (map.containsKey(need)) {
                return new int[] { map.get(need), i };
            }
            map.put(nums[i], i);
        }
        return new int[] {};
    }
}
```

### JavaScript

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
function twoSum(nums, target) {
  const map = new Map();

  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (map.has(need)) return [map.get(need), i];
    map.set(nums[i], i);
  }
  return [];
}
```

### Common mistakes

- Returning values instead of indices.
- Checking `map.has(nums[i])` before storing complement logic (wrong order).
- Using same element twice (store after check avoids this).

---

## 2. Valid Parentheses (20)

### Problem (short)

Given string `s` with `()`, `{}`, `[]` — return `true` if **valid** (every open has matching close, correct order).

```
"()"        → true
"()[]{}"    → true
"(]"        → false
"([)]"      → false
"{[]}"      → true
```

### Hint 1 — Last opened must close first

→ **Stack** (LIFO).

### Hint 2 — Matching pairs

When you see **closing** bracket, top of stack must be matching **opening**.

| Close | Expect on stack |
|-------|-----------------|
| `)` | `(` |
| `}` | `{` |
| `]` | `[` |

### Hint 3 — Algorithm

1. Stack of `char`.
2. For each `ch`:
   - If opening → push.
   - If closing → if stack empty or no match → `false`; else pop.
3. End: valid only if stack **empty**.

### Walkthrough — `"([{}])"`

```
(  push [
[  push [ ( [
{  push [ ( [ {
}  pop  → matches {
]  pop  → matches [
)  pop  → matches (
stack empty → true
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### Java

```java
class Solution {
    public boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();

        for (char ch : s.toCharArray()) {
            if (ch == '(' || ch == '{' || ch == '[') {
                stack.push(ch);
            } else {
                if (stack.isEmpty() || !matches(stack.pop(), ch)) {
                    return false;
                }
            }
        }
        return stack.isEmpty();
    }

    private boolean matches(char open, char close) {
        return (open == '(' && close == ')')
            || (open == '{' && close == '}')
            || (open == '[' && close == ']');
    }
}
```

### JavaScript

```js
/**
 * @param {string} s
 * @return {boolean}
 */
function isValid(s) {
  const stack = [];
  const pairs = { ")": "(", "}": "{", "]": "[" };

  for (const ch of s) {
    if (ch === "(" || ch === "{" || ch === "[") {
      stack.push(ch);
    } else {
      if (stack.length === 0 || stack.pop() !== pairs[ch]) return false;
    }
  }
  return stack.length === 0;
}
```

### Common mistakes

- Forgetting `stack.isEmpty()` at the end.
- Only checking count of brackets, not order (`"([)]"`).

---

## 3. Merge Two Sorted Lists (21)

### Problem (short)

Merge two **sorted** linked lists into one sorted list. Return head of merged list.

```
list1: 1 → 3 → 4
list2: 1 → 2 → 3
merged: 1 → 1 → 2 → 3 → 3 → 4
```

### Hint 1 — Dummy head node

Use `dummy` node so you don't handle empty head as special case.

```text
dummy → ... → merged list
```

### Hint 2 — Compare and attach smaller node

Pointer `curr` builds merged list:

```
while l1 != null && l2 != null:
  attach smaller node
  move that list forward
```

Then attach remaining tail (one list may still have nodes).

### Hint 3 — Visual

```
l1: 1 → 3 → 4
l2: 1 → 2 → 3

compare 1,1 → take l1 → 1
compare 3,1 → take l2 → 1 → 1
compare 3,2 → take l2 → 1 → 1 → 2
compare 3,3 → take l1 → 1 → 1 → 2 → 3
l2 done, attach l1 rest → ... → 4
```

### Complexity

| Time | Space |
|------|-------|
| O(n + m) | O(1) extra (reuse nodes) |

### Java

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;

        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                curr.next = l1;
                l1 = l1.next;
            } else {
                curr.next = l2;
                l2 = l2.next;
            }
            curr = curr.next;
        }

        curr.next = (l1 != null) ? l1 : l2;
        return dummy.next;
    }
}
```

### JavaScript

```js
/**
 * @param {ListNode} list1
 * @param {ListNode} list2
 * @return {ListNode}
 */
function mergeTwoLists(list1, list2) {
  const dummy = { val: 0, next: null };
  let curr = dummy;

  while (list1 && list2) {
    if (list1.val <= list2.val) {
      curr.next = list1;
      list1 = list1.next;
    } else {
      curr.next = list2;
      list2 = list2.next;
    }
    curr = curr.next;
  }

  curr.next = list1 || list2;
  return dummy.next;
}
```

### Recursive version (bonus)

```js
function mergeTwoLists(l1, l2) {
  if (!l1) return l2;
  if (!l2) return l1;
  if (l1.val < l2.val) {
    l1.next = mergeTwoLists(l1.next, l2);
    return l1;
  }
  l2.next = mergeTwoLists(l1, l2.next);
  return l2;
}
```

### Common mistakes

- Losing reference to head (always return `dummy.next`).
- Creating new nodes instead of re-linking (not required).
- Not attaching remaining tail after one list ends.

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | Key idea |
|---------|---------|----------|
| **Two Sum** | Hash map | Store `value → index`, look for `target - current` |
| **Valid Parentheses** | Stack | Push open; on close, pop must match |
| **Merge Sorted Lists** | Dummy node + two pointers | Pick smaller head, advance one list |

### When you see…

| Signal | Think |
|--------|-------|
| Pair / complement / two numbers | Hash map one-pass |
| Bracket matching | Stack |
| Two sorted sequences | Merge with dummy + compare |

---

*End of Day 1 LeetCode*
