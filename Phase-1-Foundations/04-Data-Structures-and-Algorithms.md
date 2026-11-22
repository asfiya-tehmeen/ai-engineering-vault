---
tags: [phase-1, dsa]
---
# 04 — Data Structures & Algorithms (DSA)

> You won't implement red-black trees at work, but DSA gives you the vocabulary for efficiency, powers many interviews, and underlies **vector search (HNSW graphs, heaps for top-k), caching (LRU), tokenizers (tries/BPE), and agent planners (graph search)**.

## 1. Complexity (Big-O)
| O | Name | Example |
|---|---|---|
| O(1) | constant | dict lookup, list index |
| O(log n) | logarithmic | binary search, heap push |
| O(n) | linear | scan a list |
| O(n log n) | linearithmic | sort |
| O(n²) | quadratic | nested loops |
| O(2ⁿ), O(n!) | exponential/factorial | brute-force subsets/permutations |

- Space complexity too. Amortized O(1) for `list.append`. Worst vs average: hash table O(1) avg, O(n) worst.
- Python specifics: `x in list` O(n); `x in set` O(1); `list.insert(0,…)` O(n) → use `collections.deque`; `sorted` = Timsort O(n log n).

## 2. Core structures
| Structure | Python | Ops |
|---|---|---|
| Array/list | `list` | index O(1), append O(1) amort |
| Hash map | `dict` | get/set O(1) avg |
| Hash set | `set` | membership O(1) |
| Stack (LIFO) | `list` | push/pop O(1) |
| Queue (FIFO) | `collections.deque` | O(1) both ends |
| Heap/priority queue | `heapq` | push/pop O(log n), peek O(1) |
| Linked list | custom | insert O(1) given node |
| Tree / BST | custom | search O(log n) balanced |
| Trie | dict-of-dicts | prefix ops O(len) |
| Graph | adjacency list `dict[node, list]` | BFS/DFS O(V+E) |
| Union-Find | arrays | near O(1) |

## 3. Algorithms & patterns (learn the *patterns*)
1. **Two pointers** (sorted arrays, palindromes)
2. **Sliding window** (substrings, max sum subarray)
3. **Hash map for lookup/counting** (two-sum, anagrams, frequency)
4. **Prefix sums**
5. **Binary search** (also on the answer: "min capacity such that…")
6. **Stack** (parentheses, monotonic stack, next greater element)
7. **Heap / top-k** (k-largest, merge k sorted lists)
8. **BFS** (shortest path unweighted, level order) / **DFS** (connected components, backtracking)
9. **Recursion & backtracking** (subsets, permutations, N-Queens)
10. **Dynamic programming** (memoization → tabulation; knapsack, LIS, edit distance, coin change)
11. **Greedy** (intervals, scheduling)
12. **Graph algos:** Dijkstra, topological sort (dependency/DAG ordering—agent workflows & build systems), union-find
13. **Sorting:** merge/quick (know how they work), counting sort
14. **Bit manipulation** (basics)

### Template: BFS
```python
from collections import deque
def bfs(graph, start):
    seen, q = {start}, deque([start])
    while q:
        node = q.popleft()
        for nxt in graph[node]:
            if nxt not in seen:
                seen.add(nxt); q.append(nxt)
    return seen
```
### Template: binary search
```python
def lower_bound(a, x):
    lo, hi = 0, len(a)
    while lo < hi:
        mid = (lo + hi) // 2
        if a[mid] < x: lo = mid + 1
        else: hi = mid
    return lo
```
### Template: DP (memoized)
```python
from functools import cache
@cache
def fib(n): return n if n < 2 else fib(n-1) + fib(n-2)
```

## 4. Why it matters for AI work (concrete)
- **Top-k similarity** → heap or partial sort.
- **Vector search** → nearest-neighbor; HNSW = layered navigable small-world *graph*.
- **LRU cache** for embeddings/LLM responses → hash map + doubly linked list (`OrderedDict`).
- **Rate limiter** → token bucket / sliding window.
- **Dedup of documents** → hashing, MinHash, sets.
- **Agent workflow DAGs** → topological sort.
- **Tokenizer (BPE)** → merge operations with priority queues/tries.

## 5. How to practice (the 150-problem plan)
- Use **NeetCode 150** (pattern-grouped) — see [[12-Resources-and-Practice-Links]].
- Daily: 1–2 problems. Method: restate → examples → brute force → optimize → code → test edge cases → state complexity.
- After each: write the *pattern* and *trap* in your notes. Re-solve after 1 week (spaced repetition).

Next → [[05-Git-CLI-Linux]]
