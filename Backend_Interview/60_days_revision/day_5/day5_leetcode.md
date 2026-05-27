# Day 4 — LeetCode

## Topics
- LRU Cache
- Insert Delete GetRandom O(1)
- Subarray Sum Equals K

---

# 1. LRU Cache

## What is LRU Cache?

LRU = Least Recently Used

If cache becomes full:
- remove least recently used item

---

## Operations

```text
get(key)  -> O(1)
put(key,value) -> O(1)
```

---

## Data Structures Used

- HashMap
- Doubly Linked List

---

## Why Doubly Linked List?

Because:
- insertion = O(1)
- deletion = O(1)

---

## Java Solution

```java
class LRUCache {

    class Node {
        int key;
        int value;
        Node prev;
        Node next;

        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    private int capacity;

    private Map<Integer, Node> map;

    private Node head;
    private Node tail;

    public LRUCache(int capacity) {

        this.capacity = capacity;

        map = new HashMap<>();

        head = new Node(0,0);
        tail = new Node(0,0);

        head.next = tail;
        tail.prev = head;
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insert(Node node) {

        node.next = head.next;
        node.prev = head;

        head.next.prev = node;
        head.next = node;
    }

    public int get(int key) {

        if (!map.containsKey(key)) {
            return -1;
        }

        Node node = map.get(key);

        remove(node);
        insert(node);

        return node.value;
    }

    public void put(int key, int value) {

        if (map.containsKey(key)) {
            remove(map.get(key));
        }

        Node node = new Node(key, value);

        insert(node);

        map.put(key, node);

        if (map.size() > capacity) {

            Node lru = tail.prev;

            remove(lru);

            map.remove(lru.key);
        }
    }
}
```

---

# 2. Insert Delete GetRandom O(1)

## Problem

Design data structure supporting:

```text
insert()
remove()
getRandom()
```

all in O(1)

---

## Idea

Use:
- ArrayList
- HashMap

---

## Java Solution

```java
class RandomizedSet {

    private List<Integer> list;
    private Map<Integer, Integer> map;
    private Random random;

    public RandomizedSet() {
        list = new ArrayList<>();
        map = new HashMap<>();
        random = new Random();
    }

    public boolean insert(int val) {

        if (map.containsKey(val)) {
            return false;
        }

        list.add(val);

        map.put(val, list.size() - 1);

        return true;
    }

    public boolean remove(int val) {

        if (!map.containsKey(val)) {
            return false;
        }

        int index = map.get(val);

        int lastElement = list.get(list.size() - 1);

        list.set(index, lastElement);

        map.put(lastElement, index);

        list.remove(list.size() - 1);

        map.remove(val);

        return true;
    }

    public int getRandom() {
        return list.get(random.nextInt(list.size()));
    }
}
```

---

# 3. Subarray Sum Equals K

## Problem

Find total subarrays having sum = k

---

## Example

```text
Input: nums = [1,1,1], k = 2
Output: 2
```

---

## Prefix Sum Idea

```text
currentSum - k
```

exists previously → valid subarray found.

---

## Java Solution

```java
class Solution {

    public int subarraySum(int[] nums, int k) {

        Map<Integer, Integer> map = new HashMap<>();

        map.put(0, 1);

        int sum = 0;
        int count = 0;

        for (int num : nums) {

            sum += num;

            if (map.containsKey(sum - k)) {
                count += map.get(sum - k);
            }

            map.put(sum, map.getOrDefault(sum, 0) + 1);
        }

        return count;
    }
}
```

---

# Pattern Cheat Sheet

| Problem | Pattern |
|---|---|
| LRU Cache | HashMap + DLL |
| Insert Delete GetRandom | ArrayList + HashMap |
| Subarray Sum Equals K | Prefix Sum + HashMap |

---

*End of Day 4 LeetCode*