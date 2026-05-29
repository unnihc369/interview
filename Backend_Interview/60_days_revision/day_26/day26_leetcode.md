# Day 26 - Linked List & Design Problems

---

# 1. Reverse Nodes in k-Group

## Problem Statement

Reverse nodes of linked list `k` at a time. If remaining nodes < k, leave as-is.

---

# Example

```text
Input:  1 → 2 → 3 → 4 → 5, k = 2
Output: 2 → 1 → 4 → 3 → 5

Input:  1 → 2 → 3 → 4 → 5, k = 3
Output: 3 → 2 → 1 → 4 → 5
```

---

# Core Idea

1. Count k nodes ahead — if insufficient, stop
2. Reverse k nodes (standard iterative reverse)
3. Connect reversed segment to rest
4. Repeat from next group

---

# Java Solution

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = new ListNode(0, head);
        ListNode groupPrev = dummy;

        while (true) {
            ListNode kth = getKth(groupPrev, k);
            if (kth == null) break;

            ListNode groupNext = kth.next;
            ListNode prev = groupNext, curr = groupPrev.next;

            while (curr != groupNext) {
                ListNode next = curr.next;
                curr.next = prev;
                prev = curr;
                curr = next;
            }

            ListNode tmp = groupPrev.next;
            groupPrev.next = kth;
            groupPrev = tmp;
        }
        return dummy.next;
    }

    private ListNode getKth(ListNode curr, int k) {
        while (curr != null && k > 0) {
            curr = curr.next;
            k--;
        }
        return curr;
    }
}
```

---

# Time Complexity

```text
O(n) — each node visited constant times
Space: O(1)
```

---

# 2. Copy List with Random Pointer

## Problem Statement

Deep copy a linked list where each node has `next` and `random` pointer.

---

# Example

```text
Node 1 → Node 2 → Node 3
random: 1→3, 2→1, 3→2
```

Return cloned list with same structure.

---

# Approach 1 — HashMap (O(n) space)

```java
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) return null;

        Map<Node, Node> map = new HashMap<>();
        Node curr = head;
        while (curr != null) {
            map.put(curr, new Node(curr.val));
            curr = curr.next;
        }

        curr = head;
        while (curr != null) {
            map.get(curr).next = map.get(curr.next);
            map.get(curr).random = map.get(curr.random);
            curr = curr.next;
        }
        return map.get(head);
    }
}
```

---

# Approach 2 — Interleaved Copy (O(1) space) — Interview Preferred

```text
Step 1: A → A' → B → B' → C → C'
Step 2: Set random pointers on copy nodes
Step 3: Separate interleaved lists
```

```java
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) return null;

        // Step 1: interleave
        Node curr = head;
        while (curr != null) {
            Node copy = new Node(curr.val);
            copy.next = curr.next;
            curr.next = copy;
            curr = copy.next;
        }

        // Step 2: random
        curr = head;
        while (curr != null) {
            if (curr.random != null) {
                curr.next.random = curr.random.next;
            }
            curr = curr.next.next;
        }

        // Step 3: separate
        Node dummy = new Node(0);
        Node copyCurr = dummy;
        curr = head;
        while (curr != null) {
            copyCurr.next = curr.next;
            copyCurr = copyCurr.next;
            curr.next = curr.next.next;
            curr = curr.next;
        }
        return dummy.next;
    }
}
```

---

# Time Complexity

```text
O(n) time, O(1) extra space (interleaved approach)
```

---

# 3. LFU Cache (Least Frequently Used)

## Problem Statement

Design cache with `get` and `put` in O(1). Evict **least frequently** used; tie-break by **least recently** used.

---

# Core Data Structures

```text
HashMap<key, Node>           — O(1) key lookup
HashMap<freq, DoublyLinkedList> — nodes at each frequency
Node: key, value, freq, prev, next
minFreq variable
```

---

# Java Solution

```java
class LFUCache {
    class Node {
        int key, val, freq;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; freq = 1; }
    }

    class DLinkedList {
        Node head = new Node(0, 0), tail = new Node(0, 0);
        int size = 0;
        DLinkedList() { head.next = tail; tail.prev = head; }
        void add(Node node) {
            node.prev = head; node.next = head.next;
            head.next.prev = node; head.next = node; size++;
        }
        void remove(Node node) {
            node.prev.next = node.next; node.next.prev = node.prev; size--;
        }
        Node removeLast() {
            if (size == 0) return null;
            Node node = tail.prev;
            remove(node);
            return node;
        }
    }

    private final int capacity;
    private int minFreq;
    private final Map<Integer, Node> keyMap = new HashMap<>();
    private final Map<Integer, DLinkedList> freqMap = new HashMap<>();

    public LFUCache(int capacity) { this.capacity = capacity; }

    public int get(int key) {
        Node node = keyMap.get(key);
        if (node == null) return -1;
        increaseFreq(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (capacity == 0) return;
        if (keyMap.containsKey(key)) {
            Node node = keyMap.get(key);
            node.val = value;
            increaseFreq(node);
            return;
        }
        if (keyMap.size() == capacity) {
            DLinkedList minList = freqMap.get(minFreq);
            Node removed = minList.removeLast();
            keyMap.remove(removed.key);
        }
        Node node = new Node(key, value);
        keyMap.put(key, node);
        freqMap.computeIfAbsent(1, k -> new DLinkedList()).add(node);
        minFreq = 1;
    }

    private void increaseFreq(Node node) {
        int f = node.freq;
        freqMap.get(f).remove(node);
        if (freqMap.get(f).size == 0 && f == minFreq) minFreq++;
        node.freq++;
        freqMap.computeIfAbsent(node.freq, k -> new DLinkedList()).add(node);
    }
}
```

---

# LFU vs LRU

| | LRU | LFU |
|---|-----|-----|
| Evict | Least recently used | Least frequently used |
| Structure | HashMap + DLL | HashMap + freq buckets + DLL |
| Best for | Temporal locality | Repeated access patterns |

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Reverse K Group | Reverse sublist + pointer math | O(n) |
| Copy Random List | HashMap or interleave | O(n) |
| LFU Cache | HashMap + freq buckets | O(1) per op |

---

# Common Interview Questions

## Q1. Why dummy node in reverse k-group?

Simplifies edge case when head changes.

## Q2. LFU tie-break rule?

Least recently used among same frequency — maintain order in DLL (add to front on access).

## Q3. LRU Cache vs LFU Cache interview frequency?

LRU more common; LFU is hard follow-up.

## Q4. O(1) for LFU put when capacity full?

Remove from `freqMap.get(minFreq)` tail — O(1) with DLL.

---

# One-Line Revision

```text
K-group reverse = segment reverse + reconnect; Random copy = map or interleave; LFU = key map + freq buckets.
```

---

*End of Day 26 LeetCode*
