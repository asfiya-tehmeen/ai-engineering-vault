---
tags: [phase-1, python, beginner]
---
# 02 — Python Core

> Use Python **3.12+** (3.13/3.14 are current). Manage with `uv` (see [[08-Testing-Tooling-Docker]]). Official tutorial: https://docs.python.org/3/tutorial/

## 1. Hello, REPL, scripts
```python
print("hello")           # run: python hello.py
# REPL: type `python` and experiment — the fastest way to learn.
```
`if __name__ == "__main__":` runs code only when the file is executed directly, not imported.

## 2. Variables & types
```python
x = 10            # int (arbitrary precision!)
y = 3.14          # float (IEEE-754 double)
s = "hi"          # str (immutable Unicode)
b = True          # bool (subclass of int)
n = None          # absence of value
z = 2 + 3j        # complex
```
- **Dynamic typing**: names have no type; objects do. `type(x)`, `isinstance(x, int)`.
- **Truthiness**: falsy = `0, 0.0, "", [], {}, set(), None, False`. Everything else truthy.
- `==` compares value; `is` compares identity (use `is None`).
- Integer ops: `/` true division, `//` floor, `%` mod, `**` power, `divmod()`.
- Floats: never compare with `==`; use `math.isclose`. For money use `decimal.Decimal`.

## 3. Strings
```python
name = "Ada"
f"Hello {name!r:>10}"            # f-strings (use these)
"a,b,c".split(",")                # ['a','b','c']
",".join(["a","b"])               # 'a,b'
s.strip(), s.lower(), s.replace("a","b"), s.startswith("x")
s[0], s[-1], s[1:4], s[::-1]      # indexing & slicing
"""multi
line"""                           # triple quotes
r"raw\nstring"                    # raw string (regex, Windows paths)
```
Strings are immutable → building in a loop with `+=` is O(n²); use `"".join(parts)`.
Learn **regex** (`re` module): `re.findall(r"\d+", s)`. Regex101.com to practice.

## 4. Collections
| Type | Mutable | Ordered | Duplicates | Use |
|---|---|---|---|---|
| `list` | ✅ | ✅ | ✅ | general sequence |
| `tuple` | ❌ | ✅ | ✅ | fixed record, dict key |
| `dict` | ✅ | ✅ (insertion) | keys unique | key→value lookup, O(1) avg |
| `set` | ✅ | ❌ | ❌ | membership, dedupe, set algebra |
| `frozenset` | ❌ | ❌ | ❌ | hashable set |

```python
nums = [3,1,2]
nums.append(4); nums.sort(); sorted(nums, reverse=True)
d = {"a":1}
d["b"] = 2; d.get("z", 0); d.items(); d.setdefault("k", [])
{**d1, **d2}  /  d1 | d2          # merge
s = {1,2}; s | {3}; s & {2}; s - {1}
```
**Comprehensions** (idiomatic Python):
```python
[x*x for x in range(10) if x%2==0]
{k: v for k, v in pairs}
{x for x in items}
(x for x in items)               # generator expression (lazy)
```
**Unpacking**: `a, b = b, a`, `first, *rest = items`, `def f(*args, **kwargs)`, `f(*lst, **dct)`.

## 5. Control flow
```python
if x > 0: ...
elif x == 0: ...
else: ...

for i, item in enumerate(items, start=1): ...
for a, b in zip(xs, ys): ...          # also zip(..., strict=True)
while cond: ...
break / continue / pass
for ... else: ...                     # else runs if no break

match command:                        # structural pattern matching (3.10+)
    case ["go", direction]: ...
    case {"type": "click", "x": x}: ...
    case _: ...
```
Ternary: `a if cond else b`. Walrus: `if (n := len(xs)) > 10:`.

## 6. Functions
```python
def greet(name: str, *, greeting: str = "Hello") -> str:
    """Docstring."""
    return f"{greeting}, {name}"
```
- Positional, keyword, default, `*args`, `**kwargs`, keyword-only (after `*`), positional-only (before `/`).
- **Gotcha:** mutable default args are evaluated once!
```python
def bad(x, acc=[]): acc.append(x); return acc     # shared across calls ❌
def good(x, acc=None):
    acc = [] if acc is None else acc; acc.append(x); return acc
```
- Functions are first-class: pass them, return them, store them. `lambda x: x+1`. `sorted(users, key=lambda u: u.age)`.
- **Scope (LEGB):** Local → Enclosing → Global → Built-in. `nonlocal`, `global`.
- **Closures**: inner function remembers enclosing variables.
- `map`, `filter`, `functools.reduce`, `partial`, `lru_cache` / `cache`.

## 7. Files, paths, serialization
```python
from pathlib import Path
p = Path("data") / "notes.txt"
text = p.read_text(encoding="utf-8")
with open(p, "w", encoding="utf-8") as f:     # context manager closes the file
    f.write("hi")
import json, csv
json.loads(s); json.dumps(obj, indent=2, ensure_ascii=False)
with open("x.csv", newline="") as f:
    for row in csv.DictReader(f): ...
```
Always specify `encoding="utf-8"`. Read huge files line by line (`for line in f`) — don't `.read()` 10 GB.
Other formats: YAML (config), TOML (`pyproject.toml`, `tomllib`), JSONL (one JSON per line — ubiquitous in LLM datasets), Parquet (columnar).

## 8. Errors & exceptions
```python
try:
    value = int(text)
except ValueError as e:
    log.warning("bad int: %s", e)
    raise                      # re-raise; or `raise CustomError(...) from e`
else:
    ...                        # no exception
finally:
    cleanup()                  # always runs
```
- Catch **specific** exceptions; never bare `except:`. Don't swallow errors silently.
- Define domain errors: `class RateLimitError(Exception): ...`.
- EAFP ("easier to ask forgiveness") vs LBYL ("look before you leap") — Python favors EAFP.
- `assert` is for invariants/dev; it can be disabled with `-O`; don't validate user input with it.

## 9. Modules & packages
```python
import math
from collections import Counter, defaultdict, deque, namedtuple
from itertools import chain, groupby, islice, product, permutations
import os, sys, re, datetime as dt, logging, argparse, subprocess, random, time, uuid
```
- A **module** = a `.py` file. A **package** = a directory with `__init__.py` (or namespace package).
- Third-party packages: `pip install` / `uv add`. Always in a **virtual environment**.
- Standard library tour (must know): `collections`, `itertools`, `functools`, `pathlib`, `json`, `re`, `datetime`, `logging`, `argparse`, `subprocess`, `sqlite3`, `concurrent.futures`, `asyncio`, `typing`, `dataclasses`, `enum`, `contextlib`, `hashlib`, `secrets`, `urllib`.

## 10. Dates, time, randomness
- Use timezone-aware datetimes: `datetime.now(timezone.utc)`. Store UTC, display local. `zoneinfo` for tz.
- `random` is **not** cryptographically secure; use `secrets` for tokens.
- Seed (`random.seed(42)`) for reproducibility — a theme in ML.

## 11. Idioms checklist
- Prefer `for x in xs` over index loops; `enumerate`, `zip`.
- Prefer comprehension over `map/filter` + lambda for readability.
- `with` for resources. `pathlib` over `os.path`. f-strings over `%`/`.format`.
- `in` on sets/dicts is O(1); on lists O(n).
- EAFP, small functions, no globals, explicit > implicit (`import this`).
- Follow **PEP 8** (use `ruff format` / `ruff check` – see Note 08).

## Mini-glossary
*iterable* (can be looped), *iterator* (has `__next__`), *hashable* (usable as dict key; immutable), *callable*, *sequence*, *mapping*, *slice*, *dunder method* (`__init__`), *PEP*, *REPL*, *virtualenv*.

Next → [[03-Python-Intermediate-to-Advanced]] · Practice → [[10-Exercises]]
