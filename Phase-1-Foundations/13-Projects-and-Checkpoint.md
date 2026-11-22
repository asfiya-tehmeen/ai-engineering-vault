---
tags: [phase-1, projects, checkpoint]
---
# 13 — Projects & Phase Checkpoint

> Rule: every project lives in its own GitHub repo with a README containing **Problem · Architecture diagram · How to run · Tests · Tradeoffs/Limitations**. This README format is what hiring managers and FDE interviewers actually want to see.

## Project 1 — `logwise` CLI log analyzer *(Notes 02, 05, 08)*
Build a CLI that ingests big log files (generate a fake 1M-line access log) and reports top endpoints, error rates, p50/p95/p99 latency, busiest minutes.
- Requirements: `argparse`/`typer`, streaming (no full load), regex parsing, `--since` filter, JSON + table output, unit tests, `pyproject.toml`, ruff in pre-commit.
- Stretch: parallelize with `multiprocessing`; compare speed.

## Project 2 — `apiwatch` resilient API client *(Notes 03, 06)*
Typed client library for a public API (e.g., GitHub or Open-Meteo): Pydantic models, pagination generator, retries with backoff+jitter honoring `Retry-After`, rate limiter, timeout config, structured logging.
- Tests with `respx`; 90% coverage on retry logic. Publish to TestPyPI as a stretch.

## Project 3 — `notekeeper` FastAPI + Postgres service *(Notes 06, 07, 08)*
CRUD notes/tags API with auth (API key → bonus: JWT), SQLAlchemy + Alembic migrations, pagination, search, rate limiting via Redis, OpenAPI docs, `docker compose` (app+db+redis), GitHub Actions CI.
- Stretch: add SSE endpoint that streams a fake token response; deploy to Cloud Run/Fly.io/Render.

## Project 4 — Messy-data pipeline *(Notes 07, 09)*
Take a messy public dataset (Kaggle "Data Cleaning Challenge" / NYC open data). Build a script: ingest → clean → validate (Pandera) → load into Postgres/DuckDB → 10 analytic SQL queries including window functions → short written insights.

## Project 5 (Capstone) — `ingestor`: concurrent API ingest service *(everything)*
Async service that pulls from 3 different public APIs on a schedule, normalizes into one schema, deduplicates, stores in Postgres, exposes a FastAPI query endpoint, handles failures gracefully (retries, dead-letter table), emits metrics/logs, runs in Docker, CI-tested.
- Must include: concurrency limit, idempotent upserts, health endpoint, config via env, a README with a **failure-modes table** (what breaks / how detected / how handled).
- This is a miniature of real FDE work: integrating messy external systems reliably.

## Self-assessment (be honest)
Rate 0 (never) – 3 (could teach it). Need ≥2 on everything before Phase 2.

| Skill | 0–3 |
|---|---|
| Write idiomatic Python (comprehensions, generators, unpacking) | |
| Debug with tracebacks, `pdb`, logging | |
| OOP/dataclasses/Protocols; choose composition vs inheritance | |
| Decorators & context managers (write my own) | |
| Type hints + mypy/pyright | |
| Pydantic models & JSON Schema | |
| asyncio: gather, semaphores, timeouts, streaming | |
| Explain GIL; pick threads vs async vs processes | |
| Arrays/hashmaps/heaps/graphs; solve LC mediums | |
| Git branching, rebase, bisect, PR workflow | |
| Shell pipelines, env vars, permissions | |
| HTTP semantics, status codes, auth, idempotency | |
| Robust API client (timeouts, retries, backoff, rate limits) | |
| Build & test a FastAPI service | |
| SQL joins, CTEs, window functions, EXPLAIN | |
| Transactions & isolation | |
| pytest fixtures/mocks/parametrize | |
| Dockerfile + compose; CI pipeline | |
| NumPy broadcasting; cosine-sim search | |
| Pandas cleaning & groupby/merge | |

## Mock interview (45 min, do with a friend or record yourself)
1. 5 min: explain a project you built and a bug you hit.
2. 20 min: one LeetCode-medium out loud.
3. 10 min: SQL question with a window function.
4. 10 min: "Design a service that ingests webhooks from a customer's system reliably" — talk about idempotency, queues, retries, signature verification, observability.

## What you've unlocked
You now have the engineering substrate to understand ML (Phase 2) and LLM systems (Phases 3–7) as *software you can build and operate*.

⏸️ **Pause point.** When you've finished (or want to start) Phase 2, tell me and I'll build it — ML & Deep Learning Foundations, with the same structure: concepts, exercises, Q&A, resources, projects.

Back to [[00-Phase-1-Index]] · [[00-Roadmap]]
