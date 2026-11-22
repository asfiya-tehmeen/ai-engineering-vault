---
tags: [phase-4, projects, checkpoint]
---
# 11 — Projects & Phase Checkpoint

> Every repo: README with **Problem · Architecture diagram · Prompts (versioned) · Eval results (with CIs) · Cost & latency table · Failure modes & mitigations · How to run**. Include a `make eval` target. This README style is what FDE hiring managers look for.

## Project 1 — `chatkit`: production-grade chat backend *(Notes 01, 04, 05)*
FastAPI service with streaming SSE endpoint, conversation store (SQLite/Postgres) keeping full content blocks, system-prompt versions, per-user rate limits and token budgets, cost meter (usage → $ by model/feature), retry/timeout policy, and structured JSON logs. Frontend: minimal HTML/JS with stop button.
**Stretch:** prompt caching on history; compaction or sliding-window summarization with fact-survival tests; provider fallback.

## Project 2 — `promptlab`: prompt test & comparison harness *(Note 02)*
YAML test cases → run N prompt versions × M models × K efforts → table of accuracy (code graders + optional LLM-judge), cost, latency, with bootstrap CIs and paired comparisons; stores results in SQLite; HTML report. Use it on three tasks (classification, extraction, summarization).
**Stretch:** automatic prompt-improvement loop with train/val split and overfitting guard.

## Project 3 — `docparse`: invoice/contract intake pipeline *(Notes 03, 06)*
Classify → per-type Pydantic schema → extraction with evidence quotes → business-rule validation (totals, dates, vendor IDs from a mock master table) → confidence scoring (dual extraction disagreement) → review queue (simple web UI showing source highlights) → idempotent write to DB → metrics dashboard.
- Dataset: public invoices/receipts (CORD/SROIE/synthetic) + 50 hand-labeled docs.
- Report: field-level precision/recall, **straight-through-processing rate**, cost/doc, failure gallery by slice (scan quality, layout, language).
**Stretch:** compare native-PDF vs page images vs OCR-first; Batch API for backfill.

## Project 4 — `sqlpilot`: safe text-to-SQL analyst *(Notes 03, 07)*
Natural-language questions over a sample warehouse (Chinook or a synthetic retail DB). Schema + glossary in context; **read-only role**; `sqlglot` validation and table allow-list; `EXPLAIN` guard; row/time limits; shows SQL + results + a chart via code execution; asks clarifying questions when ambiguous. Gold set of 50 questions → execution accuracy; red-team set for injection/destructive queries.
**Stretch:** self-correction loop on SQL errors; semantic layer/metrics definitions.

## Project 5 — `llm-gateway`: thin multi-provider gateway *(Notes 04, 07)*
Internal OpenAI-style endpoint backed by two+ providers: model aliasing (`fast`, `smart`), fallback rules, per-key budgets, request/response logging with PII redaction, response cache (exact-match) and Prometheus metrics. Include chaos tests (provider down, 429s, slow responses) proving failover and budgets.
**Stretch:** difficulty router (small → big) with measured cost–quality curve.

## Project 6 (Capstone) — `intake-copilot`: multimodal customer-ops assistant *(everything)*
Scenario (pick one): insurance claims intake, vendor onboarding (KYC docs), or HR document processing.
- Users upload PDFs/images/emails; the system classifies, extracts to schemas, **uses tools** to look up customer/policy records (mock APIs), checks business rules, drafts a reply with **citations** to policy text, and routes uncertain cases to a human with highlighted evidence.
- Engineering requirements: streaming UI, async processing queue, Batch backfill job, prompt caching on policy text, idempotent writes, audit log, cost/latency dashboards, config-driven models, eval suite in CI (30–100 labeled cases) with a pass/fail threshold.
- Deliverables: repo + 2-page **design doc** (architecture, data flow, failure modes, security/PII, cost model) + 5-minute **demo video** + **handoff runbook** (how an ops team operates it). *This mirrors real FDE deliverables.*

## Phase 4 Self-assessment (0–3; need ≥2 before Phase 5)
| Skill | 0–3 |
|---|---|
| Explain every field of a Messages request/response; handle `stop_reason` | |
| Maintain conversation state; compute cost growth per turn | |
| Handle errors/retries/rate limits correctly | |
| Know what changed in model params (sampling, thinking/effort, prefill, tool_choice) | |
| Write structured prompts with examples; run prompt experiments with metrics | |
| Prune "cruft" prompts after model upgrades | |
| Extract validated JSON (Pydantic/schema) with repair & abstention | |
| Hand-write a tool loop incl. parallel calls & errors | |
| Design tools with good descriptions/permissions; mitigate tool-based injection | |
| Stream (SSE) end to end; handle partial/cancelled streams | |
| Async fan-out with semaphore + checkpointing | |
| Use Batch API appropriately | |
| Prompt caching: structure prompts, verify via usage, debug invalidation | |
| Compaction / context editing / memory strategies; test fact survival | |
| Build a cost & latency model for a feature | |
| Send images/PDFs; use citations/Files API; build a doc pipeline w/ validation | |
| Use at least two providers behind an adapter; know switching pitfalls | |
| Choose workflow vs agent; justify the simplest tier | |
| Describe a safe text-to-SQL / tool-using design | |
| Explain model-migration rollout (shadow, canary, rollback) | |

## Mock interview (60 min)
1. 10 min — "Walk me through everything in an LLM API call, and what you log."
2. 20 min — live-code: structured extraction with Pydantic + retry-on-validation-error + `stop_reason` handling; or a tool loop with two tools.
3. 15 min — "Our chatbot bill tripled. Diagnose and fix." (usage logs, history growth, tool-result bloat, cache misses, effort/model changes, retries)
4. 15 min — design: "Process 1M scanned invoices/month with LLMs for a mid-size bank." (classification, schemas, accuracy/eval, human review, throughput, Batch vs async, PII/compliance, cost)

## What you've unlocked
You can build, run, and cost any single-model LLM feature. Phase 5 adds **knowledge**: when the model needs *your* data at scale — embeddings, vector search, chunking, hybrid retrieval, reranking, and RAG evaluation.

⏸️ **Pause point.** Tell me when you're ready for **Phase 5 — RAG & Knowledge Systems** (I'll research the current state of vector DBs, embedding models, hybrid search, GraphRAG, agentic retrieval, and RAG evals first).

Back to [[00-Phase-4-Index]] · [[../00-Roadmap]]
