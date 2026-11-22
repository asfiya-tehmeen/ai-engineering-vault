---
tags: [phase-1, qa, interview]
---
# 11 — Q&A (Phase 1)

> In Obsidian, each answer is a **collapsed callout** — click to expand. Say the answer out loud *before* expanding. Turn into flashcards with `Q::A` if you use the Spaced Repetition plugin.

## Python

> [!question]- Q1. What's the difference between a list and a tuple? When use each?
> Lists are mutable, tuples immutable (and hashable if their elements are). Use tuples for fixed records/dict keys/return of multiple values; lists for collections that change. Tuples are slightly smaller/faster.

> [!question]- Q2. Explain mutable vs immutable with an example of a bug it causes.
> Mutable objects (list, dict, set) can change in place; immutable (int, str, tuple, frozenset) can't. Bug: `def f(x, acc=[])` — the default list is created once and shared across calls. Another: `b = a` for lists makes both names point to the same object; use `a.copy()`/`copy.deepcopy`.

> [!question]- Q3. `is` vs `==`?
> `==` compares values (`__eq__`); `is` compares object identity. Use `is` for `None`/singletons. Small-int/string interning makes `is` appear to work on literals — don't rely on it.

> [!question]- Q4. What are `*args` and `**kwargs`?
> `*args` collects extra positional args into a tuple; `**kwargs` collects extra keyword args into a dict. Also used at call site to unpack. Common in wrappers/decorators to forward arguments.

> [!question]- Q5. What is a decorator? Write one.
> A callable that takes a function and returns a (usually wrapped) function, applied with `@`. Uses: retry, timing, caching, auth, registration. Always use `functools.wraps` to preserve metadata. (See note 03 for code.)

> [!question]- Q6. Generator vs list? Why use generators?
> A generator yields values lazily one at a time with O(1) memory; a list materializes everything. Use for large/streaming data (files, API pages, LLM token streams). Drawback: single-pass, no indexing/len.

> [!question]- Q7. What is the GIL and what are its consequences?
> CPython's Global Interpreter Lock allows one thread to execute Python bytecode at a time, simplifying memory management. CPU-bound threaded Python doesn't scale across cores; I/O-bound code does (GIL released during I/O). Use multiprocessing, native extensions (NumPy), or asyncio for I/O concurrency. Opt-in free-threaded builds exist in newer CPython versions.

> [!question]- Q8. asyncio vs threading vs multiprocessing?
> asyncio: single-threaded cooperative concurrency, great for thousands of I/O-bound tasks; requires async-aware libraries and never blocking the loop. Threading: preemptive, simple for blocking I/O but GIL-limited for CPU. Multiprocessing: separate processes, true parallelism for CPU-bound, higher overhead + IPC/serialization costs.

> [!question]- Q9. What does `await` actually do?
> It suspends the current coroutine and yields control to the event loop until the awaited operation completes, letting other tasks run. Forgetting `await` returns a coroutine object that never runs (a classic bug). Blocking calls (like `time.sleep`, `requests.get`) in `async def` freeze the whole loop.

> [!question]- Q10. Explain context managers.
> Objects implementing `__enter__`/`__exit__` (or `@contextmanager`) so `with` guarantees setup/teardown even on exceptions (files, locks, DB transactions, HTTP clients, tracing spans).

> [!question]- Q11. What are type hints; are they enforced?
> Annotations for humans and tools (mypy/pyright/IDE). Not enforced by the interpreter. Pydantic/FastAPI use them at runtime for validation.

> [!question]- Q12. Shallow vs deep copy?
> Shallow copies the container but shares nested objects; deep copies recursively. `copy.copy` vs `copy.deepcopy`. Matters with nested lists/dicts.

> [!question]- Q13. How does Python manage memory?
> Reference counting plus a cyclic garbage collector; objects live on a private heap managed by the interpreter (pymalloc for small objects). Leaks arise from lingering references (global caches, closures, unbounded lists).

> [!question]- Q14. What are `__init__` vs `__new__`? `__repr__` vs `__str__`?
> `__new__` creates the instance, `__init__` initializes it. `__repr__` is the unambiguous developer representation; `__str__` the readable one (falls back to repr).

> [!question]- Q15. What's a closure?
> An inner function that captures variables from its enclosing scope; they live on after the outer function returns. Basis of decorators and factories. Late-binding gotcha: lambdas in a loop capture the *variable*, not value — fix with default arg `lambda i=i: i`.

> [!question]- Q16. How would you process a 50 GB CSV on a laptop?
> Stream it: iterate by line/chunk (`pd.read_csv(chunksize=…)`), or use DuckDB/Polars lazy scans/Parquet; select only needed columns, push filters down, aggregate incrementally; avoid loading all into RAM.

> [!question]- Q17. `list.sort()` vs `sorted()`?
> `.sort()` in place, returns None; `sorted()` returns a new list from any iterable. Both Timsort, stable, accept `key=` and `reverse=`.

> [!question]- Q18. Why `if __name__ == "__main__":`?
> So code runs only when executed as a script, not on import — allowing modules to be both importable and runnable.

## DSA

> [!question]- Q19. Complexity of dict lookup, list `in`, set `in`, `list.insert(0,x)`, `sorted`?
> O(1) average; O(n); O(1) average; O(n); O(n log n).

> [!question]- Q20. How does a hash table work? What about collisions?
> A hash function maps keys to bucket indexes; collisions are handled by chaining or open addressing (Python uses open addressing with probing). Resizing keeps load factor low → O(1) average; worst case O(n). Keys must be hashable/immutable.

> [!question]- Q21. BFS vs DFS — when each?
> BFS (queue) finds shortest path in unweighted graphs and level-order; DFS (stack/recursion) for exhaustive search, cycle detection, topological ordering, connected components. Both O(V+E).

> [!question]- Q22. Explain dynamic programming.
> Break a problem into overlapping subproblems with optimal substructure; store results (memoization top-down or tabulation bottom-up) to avoid recomputation, turning exponential into polynomial.

> [!question]- Q23. How find top-k from a stream of n items?
> Maintain a min-heap of size k: O(n log k) time, O(k) memory. Quickselect gives O(n) average for in-memory arrays.

> [!question]- Q24. Design an LRU cache.
> Hash map from key→node + doubly linked list ordered by recency; get/put O(1); evict tail. In Python: `OrderedDict.move_to_end` / `popitem(last=False)`.

> [!question]- Q25. Binary search on a rotated array, or "on the answer"?
> Identify a monotonic predicate; shrink [lo, hi] by testing mid. For "min capacity to ship in D days," binary search capacity with a feasibility check.

## Git / CLI

> [!question]- Q26. merge vs rebase?
> Merge joins histories with a merge commit (preserves truth); rebase replays your commits atop another base for linear history but rewrites commits — don't rebase shared/public history.

> [!question]- Q27. `git reset` vs `revert` vs `restore`?
> `reset` moves the branch pointer (soft/mixed/hard; rewrites history); `revert` creates a new commit undoing a prior one (safe on shared branches); `restore` discards/unstages working-tree/index changes.

> [!question]- Q28. You committed an API key. What now?
> Revoke/rotate the key immediately (assume compromised), then remove from history (git filter-repo/BFG) and force-push if needed, add to `.gitignore`/use secret scanning, and check logs for misuse.

> [!question]- Q29. Explain a pipe and redirection.
> `a | b` feeds a's stdout to b's stdin; `>` writes stdout to a file, `2>` stderr, `2>&1` merges. Programs compose by being small and text-oriented.

## HTTP / APIs

> [!question]- Q30. PUT vs POST vs PATCH?
> POST creates/triggers (non-idempotent); PUT replaces a resource entirely (idempotent); PATCH partially updates.

> [!question]- Q31. What does idempotent mean and why does it matter?
> Repeating the request has the same effect as doing it once. Lets clients safely retry after timeouts. GET/PUT/DELETE are idempotent by spec; POST needs an idempotency key.

> [!question]- Q32. 401 vs 403? 429? 502 vs 504?
> 401 = not authenticated; 403 = authenticated but not permitted; 429 = rate limited (see `Retry-After`); 502 = bad gateway (invalid upstream response); 504 = gateway timed out waiting for upstream.

> [!question]- Q33. How do you design robust API client code?
> Timeouts (connect/read), retries with exponential backoff + jitter on transient errors only, honor `Retry-After`, circuit breaker, idempotency keys, response validation, structured logs with request IDs, concurrency limits, metrics.

> [!question]- Q34. How does LLM streaming work over HTTP?
> Typically Server-Sent Events: a long-lived response (`text/event-stream`) where the server emits `data: {...}` chunks incrementally; the client reads the stream and assembles deltas. Gives low time-to-first-token UX.

> [!question]- Q35. Session cookie vs JWT vs API key vs OAuth?
> Cookie = server-side session id; JWT = signed self-contained token (hard to revoke; verify signature/expiry); API key = static secret identifying an app; OAuth = delegated authorization with scoped, expiring tokens without sharing passwords.

> [!question]- Q36. How should a webhook receiver behave?
> Verify signature (HMAC) and timestamp, respond 2xx quickly, enqueue processing, be idempotent (dedupe by event id), tolerate retries/out-of-order events.

> [!question]- Q37. What is CORS?
> Browser-enforced policy restricting cross-origin requests unless the server sends permissive headers (`Access-Control-Allow-Origin` etc.). Not a server-side security control, doesn't affect curl/backends.

## SQL

> [!question]- Q38. INNER vs LEFT JOIN?
> INNER returns only matching rows; LEFT returns all left rows plus matches (NULLs for no match). Filtering right-table columns in `WHERE` can silently turn a LEFT join into inner — put such conditions in `ON`.

> [!question]- Q39. WHERE vs HAVING?
> WHERE filters rows before grouping; HAVING filters groups after aggregation.

> [!question]- Q40. What is an index and what are its tradeoffs?
> A data structure (usually B-tree) allowing fast lookups/range scans without full table scans. Costs: storage and slower writes; useless if the planner deems scans cheaper or you wrap the column in a function.

> [!question]- Q41. Explain ACID and isolation levels.
> Atomicity, Consistency, Isolation, Durability. Isolation levels trade anomalies for concurrency: Read Committed, Repeatable Read, Serializable.

> [!question]- Q42. ROW_NUMBER vs RANK vs DENSE_RANK?
> ROW_NUMBER unique sequential; RANK ties share rank with gaps after; DENSE_RANK ties share rank without gaps.

> [!question]- Q43. What's the N+1 problem?
> Fetching a list (1 query) then querying per item (N queries). Fix with JOINs, eager loading (`selectinload`), or batching.

> [!question]- Q44. How prevent SQL injection?
> Parameterized queries/prepared statements (never string-concatenate input), least-privilege DB accounts, ORM/query builders, input validation.

> [!question]- Q45. OLTP vs OLAP?
> OLTP: many small transactional reads/writes, row-oriented, normalized (Postgres). OLAP: large analytical scans/aggregations, columnar, denormalized (Snowflake/BigQuery).

## Testing / tooling

> [!question]- Q46. What should you mock and what not?
> Mock external boundaries (network, time, DB, LLM, filesystem) to keep unit tests fast/deterministic; don't mock the code under test or so much that the test checks only the mocks. Add a few integration tests with real components.

> [!question]- Q47. Docker image vs container? Why layers matter?
> Image = immutable template; container = running instance. Layers are cached — order instructions so rarely-changing steps (install deps) come before frequently-changing (copy source) to speed rebuilds.

> [!question]- Q48. VM vs container?
> VMs virtualize hardware with their own kernel (heavy, strong isolation); containers share the host kernel using namespaces/cgroups (light, fast).

> [!question]- Q49. How do you manage secrets in an app?
> Environment variables from a secret manager (not committed), least-privilege and short-lived credentials, rotation, no logging, secret scanning in CI.

> [!question]- Q50. A service works locally but fails in production. Debug approach?
> Compare environment (versions, env vars, network/IAM/firewall, resource limits), read logs/traces with request IDs, reproduce in container locally, bisect changes, check timeouts/DNS/TLS, add observability, fix root cause, add a regression test.

## Behavioral/system warm-ups
> [!question]- Q51. Walk through everything that happens when you type a URL and press Enter.
> DNS lookup → TCP handshake → TLS handshake → HTTP request → load balancer/reverse proxy → app server → DB/cache → response → browser renders (or client parses JSON). Mention caching (browser/CDN), keep-alive, HTTP/2.

> [!question]- Q52. Design a URL shortener / rate limiter (talk through).
> Clarify requirements; API; data model (id→URL); ID generation (base62 of counter / hash); storage (KV/SQL), cache hot keys, scale reads via CDN/cache, collisions, analytics async, availability. Rate limiter: token bucket in Redis keyed by user/IP, atomic Lua/INCR+TTL, return 429 + `Retry-After`.

Next → [[12-Resources-and-Practice-Links]]
