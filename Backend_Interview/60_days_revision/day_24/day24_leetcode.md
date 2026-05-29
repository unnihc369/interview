# Day 24 - Trie & Backtracking

---

# 1. Implement Trie (Prefix Tree)

## Problem Statement

Implement a trie with `insert`, `search`, and `startsWith` operations.

---

# Example

```text
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // true
trie.search("app");     // false
trie.startsWith("app"); // true
```

---

# Core Idea

Each node has:

- `children[26]` (or `Map<Character, TrieNode>`)
- `isEnd` flag marking complete word

---

# Java Solution

```java
class Trie {
    private static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }

    private final TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    public boolean search(String word) {
        TrieNode node = findNode(word);
        return node != null && node.isEnd;
    }

    public boolean startsWith(String prefix) {
        return findNode(prefix) != null;
    }

    private TrieNode findNode(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

---

# Time Complexity

```text
insert/search/startsWith: O(L) where L = word length
Space: O(total characters stored)
```

---

# 2. Word Search II

## Problem Statement

Given an `m x n` board of characters and a list of words, return all words found on the board. Each letter must be used once per word; adjacent cells (4-direction) only.

---

# Example

```text
board = [
  ['o','a','a','n'],
  ['e','t','a','e'],
  ['i','h','k','r'],
  ['i','f','l','v']
]
words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]
```

---

# Core Idea — Trie + DFS Backtracking

1. Build trie from all words; store word at end node
2. DFS from each cell
3. Prune when trie path doesn't exist
4. Mark visited, backtrack unmark

---

# Java Solution

```java
class Solution {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String word;
    }

    public List<String> findWords(char[][] board, String[] words) {
        TrieNode root = buildTrie(words);
        List<String> result = new ArrayList<>();
        int m = board.length, n = board[0].length;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                dfs(board, i, j, root, result);
            }
        }
        return result;
    }

    private TrieNode buildTrie(String[] words) {
        TrieNode root = new TrieNode();
        for (String w : words) {
            TrieNode node = root;
            for (char c : w.toCharArray()) {
                int idx = c - 'a';
                if (node.children[idx] == null) node.children[idx] = new TrieNode();
                node = node.children[idx];
            }
            node.word = w;
        }
        return root;
    }

    private void dfs(char[][] board, int i, int j, TrieNode node, List<String> result) {
        if (i < 0 || j < 0 || i >= board.length || j >= board[0].length) return;
        char c = board[i][j];
        if (c == '#' || node.children[c - 'a'] == null) return;

        node = node.children[c - 'a'];
        if (node.word != null) {
            result.add(node.word);
            node.word = null; // avoid duplicates
        }

        board[i][j] = '#';
        dfs(board, i + 1, j, node, result);
        dfs(board, i - 1, j, node, result);
        dfs(board, i, j + 1, node, result);
        dfs(board, i, j - 1, node, result);
        board[i][j] = c;
    }
}
```

---

# Time Complexity

```text
O(m * n * 4^L) worst case, heavily pruned by trie
```

---

# 3. Design Add and Search Words Data Structure

## Problem Statement

Support adding words and searching with `.` wildcard matching any single letter.

---

# Example

```text
addWord("bad")
addWord("dad")
addWord("mad")
search("pad") → false
search("bad") → true
search(".ad") → true
search("b..") → true
```

---

# Core Idea

- `addWord` → standard trie insert
- `search` → DFS; on `.` try all 26 children

---

# Java Solution

```java
class WordDictionary {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd;
    }

    private final TrieNode root = new TrieNode();

    public void addWord(String word) {
        TrieNode node = root;
        for (char c : word.toCharArray()) {
            int idx = c - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    public boolean search(String word) {
        return dfs(word, 0, root);
    }

    private boolean dfs(String word, int i, TrieNode node) {
        if (node == null) return false;
        if (i == word.length()) return node.isEnd;

        char c = word.charAt(i);
        if (c == '.') {
            for (TrieNode child : node.children) {
                if (child != null && dfs(word, i + 1, child)) return true;
            }
            return false;
        }
        return dfs(word, i + 1, node.children[c - 'a']);
    }
}
```

---

# Time Complexity

```text
addWord: O(L)
search: O(26^L) worst case with wildcards
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Implement Trie | Trie node array/map | O(L) per op |
| Word Search II | Trie + DFS backtracking | O(m*n*4^L) |
| Add Search Word | Trie + wildcard DFS | O(26^L) worst |

---

# Common Interview Questions

## Q1. Trie vs HashSet for dictionary?

Trie enables prefix search and shared prefix compression; HashSet O(1) exact lookup only.

## Q2. Why store word at trie node in Word Search II?

Avoid rebuilding word from path during DFS; enables early duplicate removal.

## Q3. Memory optimization for trie?

Use compressed trie (radix tree) or `Map` instead of fixed 26-array for sparse alphabets.

## Q4. Word Search I vs II?

I: single word DFS. II: many words — trie batches lookup and prunes search space.

---

# One-Line Revision

```text
Trie = prefix tree for O(L) ops; combine with DFS backtracking for board word search.
```

---

*End of Day 24 LeetCode*
