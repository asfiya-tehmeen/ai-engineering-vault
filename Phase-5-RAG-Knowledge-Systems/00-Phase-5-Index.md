---
tags: [phase-5, index, rag]
---
# Phase 5 — RAG & Knowledge Systems

> [!abstract] Why this phase exists
> Models don't know **your** data: private documents, last week's tickets, a customer's policies. **Retrieval-Augmented Generation (RAG)** fetches relevant knowledge at query time and puts it in the context ([[../Phase-4-Building-with-LLM-APIs/05-Context-Engineering-Caching-Cost|context engineering]]). It is the **#1 enterprise LLM pattern** and the core of most FDE engagements: "make the AI answer from *our* documents, accurately, with citations, respecting permissions."

**Duration:** 4–6 weeks · **Prereq:** Phases 1–4 (SQL/Postgres, NumPy, embeddings intuition from Phase 2, APIs & context engineering from Phase 4).

> [!warning] Freshness
> The *principles* (chunk → embed → retrieve → rerank → generate → evaluate) are stable. The *product landscape* (embedding leaderboards, vector DBs) shifts monthly and sources disagree. This phase teaches you to **evaluate on your own data** instead of trusting leaderboards or blog rankings. Check [[10-Resources-and-Practice-Links]] for live trackers.

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-RAG-Fundamentals]] | why RAG, RAG vs long context vs fine-tuning, pipeline, naive-RAG failure modes, when *not* to use RAG |
| 02 | [[02-Embeddings]] | dense/sparse/multi-vector, training, similarity metrics, choosing & evaluating models, dimensions, Matryoshka, multilingual/multimodal |
| 03 | [[03-Vector-Search-and-Databases]] | exact vs ANN, HNSW/IVF/PQ/DiskANN, filtering, pgvector, DB landscape, sizing math, multi-tenancy, ops |
| 04 | [[04-Ingestion-Parsing-and-Chunking]] | parsing PDFs/tables/OCR, cleaning, metadata, chunking strategies, contextual retrieval, parent–child, ACLs, incremental updates |
| 05 | [[05-Retrieval-and-Ranking]] | BM25, hybrid + RRF, query transformation, routing, filters, cross-encoder rerankers, late interaction, diversity, context assembly |
| 06 | [[06-Generation-and-Advanced-RAG]] | grounded prompting, citations, abstention, GraphRAG, agentic/corrective RAG, multi-hop, SQL+RAG, multimodal RAG |
| 07 | [[07-RAG-Evaluation-and-Production]] | retrieval & generation metrics, golden sets, error taxonomy, observability, security, freshness, cost, enterprise rollout |
| 08 | [[08-Build-RAG-From-Scratch]] | end-to-end implementation: parse → chunk → embed → hybrid retrieve → rerank → answer with citations → eval |
| 09 | [[09-Exercises]] | 80+ exercises |
| 10 | [[10-QA-Interview-Questions]] | 60 Q&A |
| 11 | [[11-Resources-and-Practice-Links]] | papers, docs, datasets, courses |
| 12 | [[12-Projects-and-Checkpoint]] | 6 projects + self-assessment |

## Weekly plan (5 weeks)
| Week | Focus | Output |
|---|---|---|
| 1 | Notes 01–03 | brute-force NumPy semantic search; embedding bake-off on a small labeled set |
| 2 | Notes 04, 08 | ingestion pipeline (PDF → chunks → pgvector) + naive RAG with citations |
| 3 | Note 05 | hybrid search + reranker; measure retrieval gains with recall@k/nDCG |
| 4 | Notes 06, 07 | golden set, retrieval + generation evals, error analysis; one advanced technique |
| 5 | Capstone | [[12-Projects-and-Checkpoint\|Enterprise knowledge assistant]] with permissions & eval CI |

## Completion checklist
- [ ] Explain precisely when RAG beats long-context stuffing, and when it doesn't
- [ ] Implement semantic search in NumPy and explain cosine/dot/L2
- [ ] Compare ≥3 embedding models **on your own labeled data** and pick one with evidence
- [ ] Stand up pgvector (or another vector DB) with an HNSW index, metadata filters, and tune `ef_search`/recall vs latency
- [ ] Parse messy PDFs (tables, scans) and chunk them with metadata and citations preserved
- [ ] Implement hybrid retrieval (BM25 + dense) with RRF and a cross-encoder reranker; show measured gains
- [ ] Build a golden set (≥50 queries with relevant chunks) and report recall@k, MRR, nDCG with CIs
- [ ] Evaluate generation: faithfulness, correctness, citation accuracy, abstention
- [ ] Diagnose a failing query end-to-end (parse? chunk? embed? retrieve? rerank? prompt? generate?)
- [ ] Enforce document-level permissions in retrieval
- [ ] Explain GraphRAG and agentic RAG and when each is warranted
- [ ] Capstone shipped on GitHub with an eval suite in CI

Back to [[../00-Roadmap]].
