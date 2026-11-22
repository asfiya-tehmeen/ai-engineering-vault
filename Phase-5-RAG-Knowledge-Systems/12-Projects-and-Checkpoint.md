---
tags: [phase-5, projects, checkpoint]
---
# 12 — Projects & Phase Checkpoint

> Every repo: README with **Problem · Corpus & license · Architecture diagram · Golden-set description · Results table (config × recall@k/MRR/nDCG/answer-correct/abstention/latency/cost, with CIs) · Error analysis (top failure categories and fixes) · Security notes · How to run**. A README with *numbers and failure analysis* is what separates you from tutorial-followers.

## Project 1 — `semsearch`: embeddings & ANN lab *(Notes 02, 03)*
NumPy brute-force search → FAISS Flat/HNSW/IVF-PQ → pgvector HNSW. Embedding bake-off across ≥4 models on a golden set; Matryoshka truncation and quantization study; recall–latency–memory curves; filter-selectivity experiment. Output: a decision memo ("which model/index/dims for a 5M-chunk corpus and why").

## Project 2 — `ingestor`: robust document ingestion pipeline *(Note 04)*
Connectors (folder/S3/web crawl) → parse (multi-tool routing: born-digital vs scanned vs tables) → clean/dedupe → metadata + ACLs → chunk (structure-aware, parent–child) → optional contextual enrichment → embed → idempotent upsert → deletion propagation. Queue-based with retries, DLQ, metrics, and a **parse-quality dashboard**; crash-recovery tests.

## Project 3 — `hybrid-rag`: evaluated production-style RAG *(Notes 05, 06, 08)*
Corpus of ≥500 pages (e.g., SEC filings, product docs, or a government policy set). Pipeline: hybrid (BM25 + dense) with RRF → cross-encoder rerank → query rewriting for multi-turn → grounded generation with citations and abstention → citation verification.
- **Golden set:** ≥100 queries (real + synthetic) incl. 15% unanswerable and 10% adversarial.
- **Ablation table:** dense-only → +BM25 → +rerank → +rewriting → +contextual retrieval → +parent–child; each with CIs and cost/latency.
- **Judge calibration:** compare LLM-judge to 40 human labels.
- CI gate on the golden set.

## Project 4 — `kb-agent`: agentic RAG with structured data *(Note 06)*
Agent with tools: `search_docs`, `fetch_section`, `list_outline`, `sql_query` (read-only, validated), optional `web_search`. Routes simple queries to single-shot RAG. Evaluate on 40 multi-hop/mixed (SQL + docs) questions: accuracy, tokens, latency, trajectory quality; step/budget caps; failure-mode report (loops, premature stops, over-searching).
**Stretch:** GraphRAG-lite comparison on relational questions, with indexing cost accounting.

## Project 5 — `rag-secure`: permissions, injection & privacy hardening *(Note 07)*
Take Project 3 and: (1) add multi-tenant/ACL-aware retrieval with tests for leaks (including via caches and follow-up queries); (2) red-team with prompt-injection documents and exfiltration attempts (markdown-image links, tool misuse) and mitigate; (3) PII redaction at ingestion with measured precision/recall; (4) full deletion workflow (chunks, embeddings, caches, logs) with verification; (5) audit logging; (6) a 2-page **threat model** and residual-risk statement.

## Project 6 (Capstone) — `enterprise-knowledge-assistant` *(everything)*
Pick a realistic scenario (bank policy assistant, hospital protocol assistant with synthetic data, SaaS support copilot, legal clause assistant).
**Deliverables:**
- **System:** ingestion pipeline, hybrid+rerank retrieval, routing (docs vs SQL vs "clarify"), citations with page highlights, abstention, feedback capture, multi-user permissions, observability (traces, dashboards), cost controls, eval CI (golden set ≥150 queries with SME-style labels), blue/green re-index procedure.
- **Docs for the "customer":** (a) *Data audit & scoping report*, (b) *Architecture & security overview*, (c) *Evaluation report* (metrics, slices, failure analysis, limitations), (d) *Operations runbook* (freshness, incident response, adding sources, rotating keys, re-indexing), (e) *Exec summary* with business metrics and a rollout plan.
- **Demo:** 5-minute video showing real queries, failure handling, and the metrics dashboard.
This is a **portfolio centerpiece** for FDE interviews — it mirrors the discovery → build → harden → handoff cycle.

## Phase 5 Self-assessment (0–3; need ≥2 before Phase 6)
| Skill | 0–3 |
|---|---|
| Explain RAG vs long-context vs fine-tuning with a decision rule | |
| Localize a RAG failure to a pipeline stage using traces | |
| Explain embedding training, similarity metrics, prefixes, dimensions, Matryoshka | |
| Run an embedding bake-off on my own golden set | |
| Explain HNSW/IVF/PQ and tune recall vs latency | |
| Operate pgvector (HNSW, filters, `ef_search`, EXPLAIN) | |
| Choose a vector DB with explicit criteria (incl. enterprise constraints) | |
| Parse messy PDFs/tables/scans and verify quality | |
| Chunk with structure/metadata; use parent–child and contextual enrichment | |
| Build incremental, idempotent ingestion with deletes and ACL sync | |
| Implement BM25 + dense + RRF; justify hybrid | |
| Add and evaluate a reranker (gain vs latency/cost) | |
| Apply query rewriting/multi-query/HyDE/routing appropriately | |
| Write grounded prompts with citations & abstention; verify citations | |
| Explain GraphRAG & agentic RAG trade-offs; know when not to use them | |
| Build a golden set; compute recall@k/MRR/nDCG with CIs | |
| Evaluate generation (faithfulness, correctness, citations, abstention) and calibrate a judge | |
| Know RAG metric pitfalls (faithful-but-wrong, hidden retrieval misses) | |
| Enforce permissions; defend against injection/poisoning; handle deletion | |
| Cost/latency model and enterprise rollout plan | |

## Mock interview (60 min)
1. 10 min — "Design a RAG system for 2M internal documents across 8 systems with different permissions." (ingestion, ACLs, chunking, hybrid, rerank, eval, freshness, monitoring)
2. 15 min — live: implement RRF + recall@k/MRR and explain results; or debug a failing retrieval trace.
3. 15 min — "Answer quality dropped after a re-index. Investigate." (versions, embedder/chunker changes, regressions by slice, shadow eval, rollback)
4. 10 min — "Faithfulness is 0.95 but users say answers are wrong." (retrieval recall, outdated context, parametric memory, judge calibration)
5. 10 min — customer scenario: "Our docs are scans and spreadsheets; leadership wants 100% accuracy next month." (audit, expectations, vertical slice, metrics, risk controls)

## What you've unlocked
You can connect LLMs to private knowledge reliably, measure quality honestly, and operate the system securely. Phase 6 turns retrieval and tools into **agents**: planning loops, memory, MCP servers, multi-agent patterns, sandboxing, guardrails, and agent evaluation.

⏸️ **Pause point.** Tell me when you're ready for **Phase 6 — Agents, Tools & MCP** (I'll research the current state of agent SDKs, MCP, computer-use/browser agents, multi-agent patterns, memory, sandboxing, and agent evals first).

Back to [[00-Phase-5-Index]] · [[../00-Roadmap]]
