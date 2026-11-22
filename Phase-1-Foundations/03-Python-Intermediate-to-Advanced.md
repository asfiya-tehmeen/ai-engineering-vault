---
tags: [phase-1, python, advanced]
---
# 03 — Python: Intermediate → Advanced

> These features are everywhere in AI codebases: Pydantic models, async API calls, decorators for retry/trace, generators for streaming tokens, context managers for clients.

## 1. OOP
```python
class Account:
    interest = 0.02                      # class attribute
    def __init__(self, owner: str, balance: float = 0):
        self.owner = owner               # instance attribute
        self._balance = balance          # "protected" by convention
    @property
    def balance(self): return self._balance
    @balance.setter
    def balance(self, v):
        if v < 0: raise ValueError("negative")
        self._balance = v
    def __repr__(self): return f"Account({self.owner!r}, {self._balance})"
    def __eq__(self, o): return isinstance(o, Account) and (self.owner, self._balance) == (o.owner, o._balance)
    @classmethod
    def from_dict(cls, d): return cls(**d)
    @staticmethod
    def is_valid(x): return x >= 0
```
- **Encapsulation, inheritance, polymorphism, composition.** *Prefer composition over inheritance.*
- `super().__init__()`, **MRO** (method resolution order, C3 linearization), mixins.
- **Dunder methods:** `__len__`, `__getitem__`, `__iter__`, `__enter__/__exit__`, `__call__`, `__add__`, `__hash__`, `__lt__`.
- **Abstract base classes** (`abc.ABC`, `@abstractmethod`) and **Protocols** (structural typing: "if it quacks…").
- `__slots__` saves memory; descriptors power `property`; metaclasses (rarely needed; know they exist).

## 2. dataclasses, enums, NamedTuple
```python
from dataclasses import dataclass, field
from enum import Enum

class Role(str, Enum):
    USER = "user"; ASSISTANT = "assistant"

@dataclass(frozen=True, slots=True)
class Message:
    role: Role
    content: str
    meta: dict = field(default_factory=dict)
```

## 3. Iterators & generators (critical for streaming)
```python
def read_chunks(path, size=1024):
    with open(path, "rb") as f:
        while chunk := f.read(size):
            yield chunk                  # lazy: one at a time

gen = (x*x for x in range(10**9))        # no memory blowup
```
- Iterator protocol: `__iter__` returns self; `__next__` returns next or raises `StopIteration`.
- Generators: pause/resume state; `yield from`; `send()` (coroutine-style).
- LLM streaming APIs return iterators/async iterators of token chunks.
- `itertools` for pipelines; `more-itertools` (3rd party) for extras.

## 4. Decorators & closures
```python
import functools, time

def timed(fn):
    @functools.wraps(fn)                 # keep name/docstring
    def wrapper(*a, **kw):
        t = time.perf_counter()
        try: return fn(*a, **kw)
        finally: print(f"{fn.__name__}: {time.perf_counter()-t:.3f}s")
    return wrapper

def retry(times=3, exceptions=(Exception,), backoff=0.5):   # decorator factory
    def deco(fn):
        @functools.wraps(fn)
        def wrapper(*a, **kw):
            for i in range(times):
                try: return fn(*a, **kw)
                except exceptions:
                    if i == times-1: raise
                    time.sleep(backoff * 2**i)
        return wrapper
    return deco
```
Used for: retry/backoff, caching (`@lru_cache`), tracing (`@observe`), auth, tool registration (`@tool` in agent frameworks!).

## 5. Context managers
```python
from contextlib import contextmanager
@contextmanager
def timer(label):
    t = time.perf_counter(); yield; print(label, time.perf_counter()-t)
with timer("load"): ...
```
Used for files, DB connections, locks, HTTP clients, temp dirs, tracing spans.

## 6. Type hints (modern syntax)
```python
from typing import Protocol, TypedDict, Literal, Callable, Any, Self
def top(xs: list[int], k: int = 3) -> list[int]: ...
def find(id: int) -> User | None: ...           # union with |
Handler = Callable[[str], dict[str, Any]]
class Tool(TypedDict): name: str; description: str
Mode = Literal["fast", "accurate"]
```
- Type hints are **not enforced at runtime** — checkers do: **mypy**, **pyright**, `ty`. Pydantic *does* validate at runtime.
- Generics: `T = TypeVar("T")`, or PEP 695 syntax `def first[T](xs: list[T]) -> T:` (3.12+).

## 7. Pydantic (you'll use this constantly)
```python
from pydantic import BaseModel, Field, field_validator
class Invoice(BaseModel):
    id: int
    total: float = Field(ge=0)
    currency: Literal["USD","EUR"] = "USD"
    @field_validator("id")
    @classmethod
    def positive(cls, v): 
        if v <= 0: raise ValueError("id must be > 0")
        return v

inv = Invoice.model_validate_json('{"id": 1, "total": 9.5}')
Invoice.model_json_schema()      # JSON Schema → feeds LLM structured outputs & tool definitions
```
Docs: https://docs.pydantic.dev — validation, serialization, settings (`pydantic-settings`).

## 8. Concurrency: threads, processes, asyncio
| Workload | Tool |
|---|---|
| I/O-bound (HTTP, DB) | `asyncio` or threads |
| CPU-bound (parsing, math) | `multiprocessing` / `ProcessPoolExecutor` / native libs |
| Simple parallel map | `concurrent.futures` |

**The GIL:** CPython's Global Interpreter Lock lets only one thread execute Python bytecode at a time → threads don't speed CPU-bound pure Python (they do help I/O). Free-threaded (no-GIL) builds exist from 3.13+ as an opt-in; ecosystem support is still maturing.

```python
import asyncio, httpx

async def fetch(client, url):
    r = await client.get(url, timeout=10)
    r.raise_for_status()
    return r.json()

async def main(urls):
    sem = asyncio.Semaphore(10)                  # limit concurrency (rate limits!)
    async with httpx.AsyncClient() as client:
        async def bounded(u):
            async with sem: return await fetch(client, u)
        return await asyncio.gather(*(bounded(u) for u in urls), return_exceptions=True)

asyncio.run(main(urls))
```
Concepts: event loop, coroutine, `await`, tasks (`asyncio.create_task`), `gather`, `TaskGroup` (3.11+), cancellation, timeouts (`asyncio.timeout`), `async for`, async generators, **never block the loop** (use `asyncio.to_thread` for blocking code). Async is the backbone of LLM apps: fan out 50 LLM calls in parallel with a semaphore.

## 9. Error handling & logging in real code
```python
import logging
log = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(name)s %(message)s")
log.info("processing %s", doc_id)
log.exception("failed")            # includes traceback
```
Structured (JSON) logs + request IDs make production debugging possible. Never log secrets or full PII.

## 10. Performance & profiling
- Measure first: `timeit`, `cProfile`, `py-spy`, `line_profiler`, `tracemalloc`.
- Algorithmic wins > micro-optimization. Vectorize (NumPy), batch I/O, cache, use generators.
- Memory: avoid loading everything; stream.

## 11. Packaging & project layout
```
myproj/
  pyproject.toml          # deps, tool config (PEP 621)
  src/myproj/__init__.py
  tests/
  README.md
```
Build/distribute: `uv build`, wheels, `pip install -e .`.

## 12. Advanced topics (know these exist)
- Metaclasses, descriptors, `__getattr__`, `__init_subclass__`
- `functools.singledispatch`, `typing.overload`
- Weak references, garbage collector internals
- C extensions, Cython, PyO3 (Rust ↔ Python), `ctypes`
- Subinterpreters & free-threading (newer CPython)
- Memory views, buffer protocol (zero-copy)

Next → [[04-Data-Structures-and-Algorithms]]
