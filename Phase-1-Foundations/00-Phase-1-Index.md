---
tags: [phase-1, index]
---
# Phase 1 — Software & Python Foundations

> [!abstract] Why this phase exists
> AI engineering is **software engineering with a probabilistic component**. Nearly every failure in real LLM systems (flaky API calls, bad JSON parsing, slow pipelines, un-reproducible environments, broken deploys) is a classic software problem. Skip this and everything later is built on sand. Skipping is also the #1 reason bootcamp-style AI learners can't pass interviews.

**Duration:** 6–10 weeks (faster if you already code). **Prereqs:** none.

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-How-Computers-and-Programs-Work]] | CPU/RAM/disk, processes, OS, compile vs interpret, networks basics |
| 02 | [[02-Python-Core]] | types, control flow, functions, collections, strings, files, errors, modules |
| 03 | [[03-Python-Intermediate-to-Advanced]] | OOP, iterators/generators, decorators, context managers, typing, dataclasses, Pydantic, async, GIL |
| 04 | [[04-Data-Structures-and-Algorithms]] | Big-O, arrays/hashmaps/stacks/queues/heaps/trees/graphs, patterns |
| 05 | [[05-Git-CLI-Linux]] | shell, pipes, permissions, Git workflows, GitHub, PRs |
| 06 | [[06-HTTP-APIs-JSON]] | HTTP, REST, auth, JSON, `requests`/`httpx`, FastAPI, webhooks, retries |
| 07 | [[07-SQL-and-Databases]] | relational model, SELECT/JOIN/GROUP BY, window functions, indexes, transactions, Postgres |
| 08 | [[08-Testing-Tooling-Docker]] | pytest, venvs/uv, linting, logging, Docker, CI, secrets |
| 09 | [[09-NumPy-Pandas-Data-Wrangling]] | arrays, vectorization, DataFrames, cleaning |
| 10 | [[10-Exercises]] | 60+ graded exercises |
| 11 | [[11-QA-Interview-Questions]] | 50+ Q&A with hidden answers |
| 12 | [[12-Resources-and-Practice-Links]] | courses, docs, books, problem sites |
| 13 | [[13-Projects-and-Checkpoint]] | 5 projects + self-assessment |

## Suggested weekly plan (8 weeks)
| Week | Focus | Output |
|---|---|---|
| 1 | Notes 01, 02 (first half) + CLI/Git basics | 20 small Python scripts; first GitHub repo |
| 2 | Note 02 (second half), 05 | File-processing CLI tool |
| 3 | Note 03 (OOP, generators, decorators, typing) | Refactor tool into package with types |
| 4 | Note 04 + daily problems | 40 easy/medium problems solved |
| 5 | Note 06 | Weather/news API client + small FastAPI service |
| 6 | Note 07 | Postgres/SQLite-backed app, 30 SQL problems |
| 7 | Note 08 + 09 | Tests, Dockerfile, CI on the project |
| 8 | Note 03 (async) + capstone | [[13-Projects-and-Checkpoint\|Capstone: concurrent API ingest service]] |

## Phase 1 completion checklist
- [ ] Can write a 200-line Python program from scratch, typed and tested, without copy-pasting
- [ ] Understand mutable vs immutable, references, scope, generators, decorators, context managers
- [ ] Solve LeetCode-easy in <20 min and medium in <45 min (most)
- [ ] Comfortable in a terminal; Git branching, rebasing basics, PR flow
- [ ] Can explain what happens when you `GET https://api.example.com/x` end to end
- [ ] Built & called a REST API with auth, JSON, timeouts, retries
- [ ] Write JOINs, GROUP BY, CTEs, and window functions; read a query plan
- [ ] pytest with fixtures + mocks; Dockerfile that builds & runs
- [ ] Capstone shipped on GitHub with README

> [!note] Using AI tools while learning
> Use an AI assistant to *explain* and *review*, not to write the solution first. Rule: attempt for 20 minutes → then ask for a hint, not the answer. Interviews (and real debugging) require you to hold the model in your head.

Back to [[00-Roadmap]].
