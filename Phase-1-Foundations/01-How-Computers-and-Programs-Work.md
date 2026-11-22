---
tags: [phase-1, concepts, beginner]
---
# 01 — How Computers & Programs Work

*Goal: build the mental model every later concept hangs on. Beginner → a few advanced notes at the end.*

## 1. The machine
- **CPU** executes instructions (billions/sec). Has registers and small fast caches (L1/L2/L3).
- **RAM** is fast, volatile working memory. Your variables live here. Lose power → gone.
- **Disk (SSD/HDD)** is slower, persistent. Files, databases.
- **GPU** has thousands of simple cores — great at the same math repeated over many numbers (matrix multiplication). This is why GPUs power deep learning (Phase 2–3). GPU has its own memory (**VRAM**), and "does the model fit in VRAM?" becomes a central question later.
- **Network** connects machines; much slower than RAM and with *failures*. Latency numbers every engineer should know (approx.): L1 cache ~1 ns, RAM ~100 ns, SSD ~100 µs, same-datacenter round trip ~0.5 ms, cross-continent ~100+ ms. An LLM API call is typically **hundreds of ms to tens of seconds**.

> [!example] Intuition
> RAM is your desk, disk is your filing cabinet, network is mailing a letter. Doing a network call inside a loop of 10,000 iterations is mailing 10,000 letters.

## 2. Bits, bytes, text
- Everything is bits. 8 bits = 1 byte. Integers, floats (IEEE-754 – hence `0.1 + 0.2 != 0.3`), text.
- **Text encoding:** ASCII → **Unicode** (code points) → **UTF-8** (variable-width byte encoding, the web standard). A Python `str` is Unicode text; `bytes` is raw bytes. `"é".encode("utf-8")` → 2 bytes. Mixing them up causes `UnicodeDecodeError` — a very common bug when parsing files/PDFs.
- **Tokens** (Phase 3) are *not* characters or words; they're chunks produced by a tokenizer. Know now that LLMs read integers, not text.

## 3. Programs, processes, threads
- A **program** is code on disk. A **process** is a running instance with its own memory space. A **thread** is a unit of execution inside a process sharing memory.
- The **operating system (OS)** schedules processes, manages memory, files, network, permissions.
- **Environment variables** are key/value config passed to a process (`OPENAI_API_KEY`, `PATH`). API keys go here, never in code.
- **stdin / stdout / stderr** — the three standard streams; the basis of piping in the shell.
- **Exit codes** — `0` = success, non-zero = failure; CI relies on this.

## 4. Compiled vs interpreted; Python's execution
- *Compiled* (C, Rust, Go): source → machine code ahead of time; fast.
- *Interpreted* (Python, JS): an interpreter executes code more dynamically; slower per operation, faster to write.
- CPython compiles source to **bytecode** (`.pyc`) and runs it on a virtual machine. Python is **dynamically typed** (types attach to values at runtime) and **garbage collected** (reference counting + cycle collector).
- Why Python dominates AI: the heavy math runs in **C/C++/CUDA** libraries (NumPy, PyTorch); Python is just the "glue".

## 5. Memory model essentials
- **Stack** (function call frames, local vars) vs **heap** (objects). In Python all objects live on the heap; variables are *names bound to references*.
- **Pass by object reference**: functions receive a reference to the same object. Mutating a list inside a function affects the caller's list; rebinding doesn't.
- **Memory leaks** in long-running services (e.g., unbounded caches, growing chat histories) are a real production issue.

## 6. Networking in 10 minutes
- **IP address** locates a machine; **port** locates a service on it; **DNS** maps names → IPs.
- **TCP**: reliable, ordered byte stream (HTTP/1.1, HTTP/2 run on it). **UDP**: unreliable, fast (video, DNS, QUIC/HTTP-3 builds on it).
- **TLS/HTTPS**: encryption + server identity via certificates.
- **Client/server model**: client sends a request, server replies. Servers are *stateless* by default — state lives in a DB/cache.
- **Latency vs throughput**: time for one thing vs things per second. LLM apps care about both (time-to-first-token vs tokens/sec).

## 7. Core ideas used all career long
| Concept | One-liner |
|---|---|
| Abstraction | hide complexity behind an interface |
| Idempotency | doing it twice = doing it once (critical for retries) |
| Determinism | same input → same output (LLMs are *not* deterministic by default!) |
| State | data that persists between operations |
| Side effect | change outside the function's return value |
| Concurrency vs parallelism | interleaving tasks vs literally simultaneous |
| Caching | trade memory for speed; hardest part is invalidation |
| Failure is normal | networks drop, APIs 500, disks fill — design for it |

## 8. Advanced notes
- **Virtual memory & paging**, **memory-mapped files** (used to load huge model weights lazily).
- **System calls**: how user programs ask the kernel for I/O. `strace` is a superpower for debugging.
- **Containers vs VMs**: containers share the host kernel and isolate with namespaces/cgroups; VMs virtualize hardware. (Docker – see [[08-Testing-Tooling-Docker]].)
- **CAP theorem / eventual consistency** — will appear in distributed systems & vector DBs.
- **Big-endian/little-endian**, **floating point formats** (fp32/fp16/bf16/int8) – critical in Phase 3/9 for quantization.

## Self-check
- Why can a list passed to a function be modified for the caller?
- Why is `bytes` ≠ `str`?
- Why is a network call in a tight loop a smell?
- Why must retries only happen on idempotent operations?

Next → [[02-Python-Core]]
