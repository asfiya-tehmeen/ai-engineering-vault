---
tags: [phase-1, exercises]
---
# 10 — Exercises (Phase 1)

Legend: 🟢 easy · 🟡 medium · 🔴 hard. Tick the box when solved **without looking at the answer**. Write each solution in a repo `phase1-exercises/`. Add tests (pytest) for 🟡/🔴.

## A. Python basics (Note 02)
- [ ] 🟢 A1. FizzBuzz 1–100, then generalize to any list of (divisor, word) rules.
- [ ] 🟢 A2. Read a text file; print the 10 most frequent words (case-insensitive, strip punctuation). Use `Counter`.
- [ ] 🟢 A3. Reverse a string 3 ways (slice, loop, `reversed`). Check palindromes ignoring spaces/case.
- [ ] 🟢 A4. Convert a list of Celsius temps to Fahrenheit via comprehension; filter > 80°F.
- [ ] 🟢 A5. Merge two dicts; sum values for duplicate keys.
- [ ] 🟢 A6. Flatten a nested list of arbitrary depth (recursion).
- [ ] 🟡 A7. Write `is_valid_email(s)` with regex; list 5 edge cases it gets wrong and why regex isn't a perfect validator.
- [ ] 🟡 A8. Parse a CSV of transactions → dict of `{category: total}`. Handle malformed rows with logging, don't crash.
- [ ] 🟡 A9. Implement the mutable-default-arg bug, then fix it. Explain in a comment.
- [ ] 🟡 A10. Write a CLI (`argparse`) `wc`-clone: lines/words/chars for files; support stdin.
- [ ] 🟡 A11. Caesar cipher encrypt/decrypt with shift; unit tests.
- [ ] 🟡 A12. Group anagrams from a word list (`defaultdict(list)` + sorted key).
- [ ] 🔴 A13. Write a tokenizer for arithmetic expressions and evaluate with correct precedence (shunting-yard) — no `eval`.
- [ ] 🔴 A14. Implement a minimal JSON parser (objects, arrays, strings, numbers, bool, null). Compare to `json.loads` on 20 samples.

## B. Intermediate Python (Note 03)
- [ ] 🟢 B1. Create a `Vector2D` class with `__add__`, `__mul__`, `__repr__`, `__eq__`, `__abs__`.
- [ ] 🟢 B2. Convert B1 into a frozen `dataclass`.
- [ ] 🟡 B3. Write a `@timed` decorator and a `@retry(times, backoff)` decorator factory; test retry with a function that fails twice then succeeds.
- [ ] 🟡 B4. Write a generator `read_jsonl(path)` that yields parsed dicts lazily and skips bad lines (log them). Process a 1M-line file with flat memory.
- [ ] 🟡 B5. Write a context manager `temp_chdir(path)` both via class and `@contextmanager`.
- [ ] 🟡 B6. Implement an LRU cache class using `OrderedDict`; compare with `functools.lru_cache`.
- [ ] 🟡 B7. Define Pydantic models for a nested "Order with line items"; validate good and bad payloads; export JSON Schema.
- [ ] 🟡 B8. Add type hints to a 100-line script; get `mypy --strict` (or pyright) to pass.
- [ ] 🟡 B9. Write a `Protocol` `Storage` with `get/put`, and two implementations (in-memory, file). Code to the protocol.
- [ ] 🔴 B10. Build a mini plugin registry: `@register("name")` decorator populates a dict; dispatch by name. (This is how agent `@tool` decorators work.)
- [ ] 🔴 B11. Implement a thread-safe token-bucket rate limiter; test with threads.
- [ ] 🔴 B12. Write an `async` function that fetches 100 URLs (use `httpbin.org/delay/1`) with concurrency limit 10; prove total time ≈ 10 s not 100 s. Add per-request timeout and retry.
- [ ] 🔴 B13. Write an async generator that simulates streaming tokens with delays, and an `async for` consumer that prints them live.
- [ ] 🔴 B14. Demonstrate the GIL: CPU-bound task with threads vs processes; measure and explain.

## C. DSA (Note 04) — do these first, then the NeetCode list
- [ ] 🟢 C1. Two Sum (hash map). 🟢 C2. Valid Parentheses (stack). 🟢 C3. Best Time to Buy/Sell Stock.
- [ ] 🟢 C4. Binary Search. 🟢 C5. Merge Two Sorted Lists. 🟢 C6. Valid Anagram.
- [ ] 🟡 C7. Longest Substring Without Repeating Characters (sliding window).
- [ ] 🟡 C8. Top K Frequent Elements (heap / bucket).
- [ ] 🟡 C9. Number of Islands (DFS/BFS). 🟡 C10. Course Schedule (topological sort).
- [ ] 🟡 C11. Group Anagrams. 🟡 C12. Product of Array Except Self. 🟡 C13. Merge Intervals.
- [ ] 🟡 C14. LRU Cache (LeetCode 146). 🟡 C15. Coin Change (DP).
- [ ] 🔴 C16. Word Ladder (BFS). 🔴 C17. Median of Data Stream (two heaps). 🔴 C18. Edit Distance (DP).
- [ ] 🔴 C19. Implement Trie, then autocomplete top-k. 🔴 C20. Dijkstra on a weighted grid.

## D. Shell & Git (Note 05)
- [ ] 🟢 D1. One-liner: top 10 most common words in a file using `tr`, `sort`, `uniq`, `head`.
- [ ] 🟢 D2. Create repo, 3 branches, merge one cleanly, resolve one deliberate conflict.
- [ ] 🟡 D3. Use `git rebase -i` to squash 4 messy commits into 1 clean commit.
- [ ] 🟡 D4. Break a repo on purpose; find the breaking commit with `git bisect`.
- [ ] 🟡 D5. "Lose" a commit with `reset --hard`; recover it with `reflog`.
- [ ] 🟡 D6. Write a bash script that backs up a dir to a timestamped tar.gz with `set -euo pipefail`.
- [ ] 🔴 D7. Complete OverTheWire Bandit levels 0–15.
- [ ] 🔴 D8. Open a real PR to an open-source repo (docs fix is fine, find a `good first issue`).

## E. HTTP & APIs (Note 06)
- [ ] 🟢 E1. With `curl`, hit `httpbin.org` for GET/POST/headers/status/delay; inspect with `-v`.
- [ ] 🟢 E2. Fetch from a public API (e.g., JSONPlaceholder, Open-Meteo weather — no key) and print a formatted table.
- [ ] 🟡 E3. Write `get_with_retry()` using httpx: timeouts, backoff+jitter, honors `Retry-After`, retries only on 429/5xx. Test with `respx` mocks.
- [ ] 🟡 E4. Build a paginated-API client that exposes a generator over all items (cursor pagination).
- [ ] 🟡 E5. Build a FastAPI CRUD service with Pydantic models, validation errors (422), and tests via `TestClient`.
- [ ] 🟡 E6. Add API-key auth as a FastAPI dependency; return 401/403 correctly.
- [ ] 🔴 E7. Implement an SSE endpoint streaming 20 "tokens" at 100 ms intervals; write a client that consumes it with httpx.
- [ ] 🔴 E8. Webhook receiver: verify HMAC-SHA256 signature, reject replays (timestamp window), dedupe via idempotency key, ack fast and enqueue work.
- [ ] 🔴 E9. OAuth2 auth-code flow against GitHub (register an OAuth app) — read your profile.

## F. SQL (Note 07) — use SQLite/Postgres + the sites in [[12-Resources-and-Practice-Links]]
- [ ] 🟢 F1. Design schema for blog: users, posts, comments, tags (many-to-many). Draw ER diagram.
- [ ] 🟢 F2. 10 SELECT/WHERE/ORDER BY queries on a sample DB.
- [ ] 🟡 F3. Top 3 customers by revenue per month (window function + CTE).
- [ ] 🟡 F4. Users who signed up but never ordered (anti-join, 3 ways).
- [ ] 🟡 F5. Running total & 7-day moving average of daily sales.
- [ ] 🟡 F6. Find duplicate emails; delete duplicates keeping earliest.
- [ ] 🟡 F7. `EXPLAIN ANALYZE` a slow query on 1M rows; add an index; record before/after.
- [ ] 🔴 F8. Recursive CTE: org chart – all reports (direct + indirect) of a manager.
- [ ] 🔴 F9. Sessionization: group user events into sessions with a 30-minute gap (LAG + running sum).
- [ ] 🔴 F10. Demonstrate a lost update / non-repeatable read with two psql sessions, then fix via isolation level or `FOR UPDATE`.
- [ ] 🔴 F11. Install pgvector; insert 10k random vectors; run nearest-neighbor query with and without an HNSW index; compare time.

## G. Testing, Docker, CI (Note 08)
- [ ] 🟢 G1. Write 10 pytest tests incl. parametrize + fixture for a utility module.
- [ ] 🟡 G2. Mock an HTTP dependency; assert your retry logic's behavior.
- [ ] 🟡 G3. Dockerize the FastAPI service; image < 200 MB; run as non-root; add healthcheck.
- [ ] 🟡 G4. `docker compose` with app + Postgres + Redis; app reads config via env vars.
- [ ] 🟡 G5. GitHub Actions: lint + type-check + test on every PR; fail on <80% coverage.
- [ ] 🔴 G6. Multi-stage Dockerfile; compare image sizes; explain layer caching with timing evidence.
- [ ] 🔴 G7. Deploy the container to a free tier (Cloud Run / Fly.io / Render / Railway) and hit it publicly.

## H. NumPy / Pandas (Note 09)
- [ ] 🟢 H1. Implement mean/std/softmax in pure Python then NumPy; benchmark on 10⁶ numbers.
- [ ] 🟡 H2. Cosine-similarity top-k search over 100k random 384-d vectors; time brute force.
- [ ] 🟡 H3. Clean a messy CSV (dupes, mixed date formats, currency strings, nulls) → validated Parquet.
- [ ] 🟡 H4. Re-do F3 in pandas **and** DuckDB; compare.
- [ ] 🔴 H5. Explain with a worked example why `(3,1)+(4,)` broadcasts to `(3,4)`; implement batched pairwise distance without loops.

## I. Debugging drills
- [ ] 🟡 I1. Take a buggy script (ask a friend or write 5 subtle bugs: off-by-one, shared default list, shadowed builtin, timezone naive/aware compare, float equality) and find all with `pdb`/logging.
- [ ] 🟡 I2. Read a stack trace top-to-bottom aloud for 5 different exceptions; say what each means before googling.

## Weekly cadence
Mon–Thu: 1 concept note + 3–4 exercises. Fri: DSA catch-up. Sat: project work. Sun: review Q&A in [[11-QA-Interview-Questions]].
