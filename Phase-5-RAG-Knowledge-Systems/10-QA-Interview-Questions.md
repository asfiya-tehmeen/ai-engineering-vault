---
tags: [phase-5, qa, interview]
---
# 10 — Q&A (Phase 5)

> Answer aloud first, then expand.

## Fundamentals
> [!question]- Q1. What is RAG and what problems does it solve?
> Retrieving relevant external knowledge at query time and placing it in the model's context so answers are grounded in current/private data with citations. Addresses knowledge cutoffs, private data access, hallucination on long-tail facts, and provenance.

> [!question]- Q2. RAG vs long-context vs fine-tuning?
> RAG for large/changing/permissioned corpora and citations; long-context (with caching) for a single bounded document or small corpus; fine-tuning for behavior/style/format and domain language, not for injecting facts. Real systems combine them.

> [!question]- Q3. Describe the RAG pipeline and where it typically fails.
> Parse → clean → chunk → enrich → embed → index; at query: rewrite/route → retrieve (hybrid, filters) → rerank → assemble context → generate with citations → validate. Most failures are upstream: parsing, chunking, retrieval misses, ranking, context assembly — not the LLM.

> [!question]- Q4. A RAG answer is wrong. How do you debug?
> Localize the first broken stage: is the answer in the corpus? parsed correctly? chunked intact? retrieved in top-k? ranked high enough to enter context? present in the prompt? used correctly by the model? Inspect traces at each hop, tag the failure, fix the smallest thing, re-run the golden set.

> [!question]- Q5. When is plain RAG the wrong tool?
> Aggregations and numeric questions (use SQL/APIs), exact ID lookups (database/keyword), whole-document reasoning (long-context/map-reduce), tiny static corpora (just stuff + cache), or global thematic questions (hierarchical summaries/GraphRAG).

## Embeddings
> [!question]- Q6. How are embedding models trained?
> Mostly contrastively: pull relevant (query, passage) pairs together, push negatives apart (InfoNCE with in-batch and hard negatives), often multi-stage on large mined/synthetic data, frequently starting from an LLM or encoder backbone with pooling and normalization.

> [!question]- Q7. Cosine vs dot product vs L2?
> For normalized vectors they give equivalent rankings. Use the metric the model was trained with; normalize if unsure; index with the same metric you query with.

> [!question]- Q8. Why might dense retrieval miss "ERR-4021"?
> Embeddings capture semantics, compressing exact token identity; rare identifiers/codes aren't reliably represented. Hybrid search with BM25 or exact filters fixes this.

> [!question]- Q9. How do you choose an embedding model?
> Shortlist from MTEB (retrieval task, your language/size/license), then evaluate on your own golden set across slices, considering latency, cost, dimensions, max input length, privacy/license. Public scores are informative but not decisive — and sources disagree on leaders.

> [!question]- Q10. What's Matryoshka representation learning?
> Training so that prefixes of the vector are themselves good embeddings; you can truncate (then renormalize) to cut storage/latency with modest accuracy loss.

> [!question]- Q11. Dense vs sparse vs multi-vector?
> Dense: one semantic vector per text. Sparse: term-weight vectors (BM25/SPLADE) good for exact/rare terms. Multi-vector (ColBERT): a vector per token with MaxSim scoring — finer matching at higher storage cost. Hybrids combine them.

> [!question]- Q12. What happens if you change embedding models?
> Vectors from different models are incompatible; you must re-embed and re-index the whole corpus, re-evaluate, and ideally do a blue/green rollout.

> [!question]- Q13. Why use a query prefix/instruction for some embedders?
> Asymmetric retrieval models are trained with specific formats for queries vs passages; omitting them degrades recall.

## Vector search
> [!question]- Q14. Exact vs approximate nearest neighbor?
> Exact computes all distances (accurate, O(N·d)); ANN indexes (HNSW, IVF, PQ, DiskANN) trade a little recall for far lower latency at scale. Measure recall vs exact as ground truth.

> [!question]- Q15. Explain HNSW.
> A multi-layer proximity graph: search enters at a sparse top layer, greedily descends to denser layers, and explores neighbors in the bottom layer. Parameters: M (connectivity), ef_construction (build quality), ef_search (query breadth: recall vs latency). Memory-hungry; good incremental inserts.

> [!question]- Q16. Pre-filtering vs post-filtering with ANN?
> Post-filtering can return too few/no results when filters are selective; pre-filtering is exact but can be slow; modern DBs integrate filtering into traversal — benchmark under your filter selectivity.

> [!question]- Q17. When pgvector vs a dedicated vector DB?
> pgvector when you already run Postgres and scale/filter needs are moderate (relational joins, transactions, simple ops). Dedicated/managed DBs when scale, multi-tenancy, filtering performance, or ops simplicity demand it, or the customer's platform dictates.

> [!question]- Q18. Estimate memory for 20M × 768-d fp16 vectors.
> 20M × 768 × 2 B ≈ 30.7 GB raw, plus index overhead (e.g., HNSW graph) — plan for ~1.5–2×, so ~45–60 GB; quantization or disk-based indexes can reduce this.

> [!question]- Q19. How do you handle updates and deletes?
> Idempotent upserts keyed by (doc_id, content hash), delete old chunks when documents change, propagate deletions (including embeddings, caches, logs for privacy requests), periodic reindex/vacuum, and versioned blue/green rebuilds.

> [!question]- Q20. How do you guarantee tenant isolation?
> Enforce tenant/ACL filters (or separate indexes/partitions) in the retrieval layer, test with adversarial users, default-deny, audit logs; never rely on the LLM to withhold content.

## Ingestion & chunking
> [!question]- Q21. Why is chunking so important?
> Retrieval granularity should match question granularity: too small loses context; too large dilutes embeddings and adds noise. Bad chunk boundaries split answers across chunks and are a leading cause of retrieval failure.

> [!question]- Q22. Chunking strategies and a sensible default?
> Fixed-size, recursive/structure-aware, semantic, sentence-window, parent–child, hierarchical, late chunking, proposition-based. Default: structure-aware splitting at ~300–600 tokens with small overlap, heading-path prefixes, tables kept intact — then measure alternatives on your golden set.

> [!question]- Q23. What is contextual retrieval?
> Using an LLM to write a short chunk-specific context (document/section situating the chunk) and prepending it before embedding and BM25 indexing, so chunks that say "the company's revenue grew 3%" carry their who/when. Reported to reduce retrieval failures substantially, especially with hybrid + reranking; cost is manageable with prompt caching/batching.

> [!question]- Q24. What is parent–child (small-to-big) retrieval?
> Embed small chunks for precise matching but return the larger parent section for generation, giving precision plus context.

> [!question]- Q25. How do you parse tables?
> Preserve structure (Markdown/HTML/CSV with captions and headers), or use vision/layout parsers; for numeric questions, load into a structured store and use SQL/code with the table cited.

> [!question]- Q26. How do you handle permissions at ingestion?
> Capture source-system ACLs per document, sync permission changes (even without content changes), store allowed principals as metadata, and filter at retrieval by the requester's identity.

> [!question]- Q27. How would you make ingestion incremental and reliable?
> Change detection (webhooks/CDC/hash compare), queue-based idempotent stages with retries and dead-letter queues, content-hash keys, versioned artifacts, observability on throughput/failures/cost, and re-index procedures.

## Retrieval & ranking
> [!question]- Q28. Why hybrid search?
> Dense captures semantics/paraphrase; BM25 captures exact terms and rare tokens. Fusing them is more robust across query types.

> [!question]- Q29. Explain Reciprocal Rank Fusion.
> Score each document by Σ 1/(k + rank) across retrievers (k≈60). Rank-based, so no score normalization needed; robust default fusion.

> [!question]- Q30. Bi-encoder vs cross-encoder?
> Bi-encoders embed query/doc independently (precomputable, fast, coarse); cross-encoders jointly encode query+doc (slower, more accurate), so they rerank a small candidate set.

> [!question]- Q31. How do you decide if a reranker is worth it?
> A/B on the golden set: gain in nDCG/recall@k and end-to-end correctness versus added latency and cost at chosen candidate sizes. Results vary by corpus; some benchmarks show little gain over strong hybrid.

> [!question]- Q32. What is HyDE and when can it hurt?
> Generate a hypothetical answer and embed that instead of the query. Helps vague/short queries; can hurt when the generated text hallucinates specifics that steer retrieval away from the truth.

> [!question]- Q33. Why rewrite queries in multi-turn chat?
> Follow-ups depend on history ("and for contractors?"); retrieval needs a standalone query, so rewrite with context.

> [!question]- Q34. What is MMR and why use it?
> Maximal Marginal Relevance picks results maximizing relevance while penalizing redundancy to existing picks, improving diversity and coverage.

> [!question]- Q35. How do you order and assemble context?
> Label sources, dedupe, expand with parent/neighbor chunks within a token budget, place the strongest evidence first/last to mitigate lost-in-the-middle, and limit k adaptively; more context isn't automatically better.

> [!question]- Q36. How does routing help?
> A router picks the right strategy per query — no retrieval, RAG on a specific index, SQL/API, web search, or a clarifying question — improving quality and cost; evaluate the router separately.

## Advanced RAG
> [!question]- Q37. What is GraphRAG and what are its trade-offs?
> LLM extracts entities/relations into a knowledge graph, detects communities and summarizes them; queries use graph traversal (local) or community summaries (global). Great for multi-hop/global sensemaking; costs include heavy index-time LLM usage, extraction errors, maintenance, and latency. Common consensus: graphs complement vectors, not replace them; validate against a strong hybrid baseline.

> [!question]- Q38. What is agentic RAG?
> Retrieval exposed as tools in an LLM loop that plans, searches, reads, reflects, and iterates. Better for multi-hop/ambiguous queries; costs more latency/tokens and is harder to evaluate; use routing so simple queries take the cheap path and cap steps/budget.

> [!question]- Q39. Corrective RAG / Self-RAG in one line each?
> CRAG grades retrieved documents and corrects (rewrite/fallback search) when poor; Self-RAG trains the model to emit reflection tokens deciding when to retrieve and to critique outputs.

> [!question]- Q40. How do you answer multi-hop questions?
> Decompose into sub-questions, retrieve iteratively using intermediate findings, or use graph links; verify combined evidence; watch for error propagation across hops.

> [!question]- Q41. How would one assistant handle both policies and sales numbers?
> A router/agent chooses between document RAG and text-to-SQL/API tools (with a semantic layer, read-only access, validation), then composes an answer citing both document passages and query results.

> [!question]- Q42. How do you produce reliable citations?
> Stable chunk IDs and human-readable sources, native/verified citation features where available, post-hoc verification that quoted spans exist and support claims, and metrics for citation precision/recall.

## Evaluation & production
> [!question]- Q43. How do you build a RAG golden set?
> Collect real questions across query types (including unanswerable and adversarial), have SMEs label relevant evidence at doc/section level with reference answers or key facts, measure agreement, split dev/test, version it, and keep adding production failures. Use synthetic queries for scale but beware their optimism bias.

> [!question]- Q44. Which retrieval metrics and why?
> Recall@k (ceiling for answer quality), MRR/nDCG (ranking quality), hit rate, context precision, plus latency/cost — with bootstrap CIs and slice analysis.

> [!question]- Q45. Which generation metrics?
> Correctness vs reference, faithfulness/groundedness, answer relevance, completeness, citation precision/recall, and abstention accuracy (false-answer vs false-abstain).

> [!question]- Q46. Why can faithfulness be high even when retrieval failed?
> Faithfulness measures support by the retrieved context, not truth; a fluent answer from the model's parametric memory or from wrong/outdated context can look faithful. Measure retrieval recall independently and pair faithfulness with correctness.

> [!question]- Q47. How reliable are LLM judges for RAG?
> Useful but biased (verbosity, position, self-preference) and prompt/model-dependent. Calibrate against human labels, use rubrics with evidence quotes, pin judge versions, use pairwise comparisons, and re-validate when changing the judge.

> [!question]- Q48. How do you prevent prompt injection through retrieved documents?
> Treat documents as untrusted data: delimit and label them, instruct to ignore embedded instructions, restrict tool permissions (no sensitive actions without confirmation), validate/filter outputs and rendering, separate reader and actor roles, and monitor.

> [!question]- Q49. How do you enforce access control?
> Filter retrieval by the authenticated user's groups using ACL metadata synced from source systems, default-deny, test for leakage (including caches), audit access; optionally per-tenant indexes.

> [!question]- Q50. How do you keep an index fresh?
> Define freshness SLAs, change detection and incremental ingestion, deletion propagation, versioned documents with effective dates, monitoring of ingestion lag, and blue/green re-indexing for pipeline changes.

> [!question]- Q51. A customer asks to delete a user's data. What must you remove?
> Source docs, parsed artifacts, chunks, embeddings/index entries, caches (semantic/retrieval/answer), logs/traces containing the data, and backups per policy — and verify with tests.

> [!question]- Q52. Estimate the cost per query of a RAG system.
> Query embedding + retrieval + rerank + rewrite/judge calls + LLM input (system + context) and output tokens; LLM tokens usually dominate. Reduce with smaller context, caching, routing, smaller auxiliary models; validate with logged usage.

> [!question]- Q53. How do you improve a RAG system's accuracy from 70% to 85%?
> Build/refresh the golden set, do error analysis, fix top categories (parsing/chunking/metadata first), add hybrid + reranking, query rewriting, contextual enrichment, abstention thresholds, prompt refinements, and iterate with measured gains and no regressions.

> [!question]- Q54. Users say answers are "sometimes outdated." Diagnose.
> Check document versions/effective dates, ingestion lag/failed updates, duplicate old versions in the index, ranking ignoring recency, stale caches; add version metadata, recency boosts/filters, delete superseded docs, freshness monitoring.

> [!question]- Q55. How do you evaluate an agentic RAG system?
> Final answer quality plus trajectory metrics: number/quality of searches, redundancy, evidence coverage, step/cost/latency distributions, failure modes (loops, premature stopping), with step and budget limits tested.

## FDE scenarios
> [!question]- Q56. Customer data is a mess (scans, duplicates, no metadata). First two weeks?
> Data audit on a sample, parse-quality review, dedupe/version strategy, define metadata/ACL needs, build a golden set with SMEs, deliver a thin vertical slice on one source to show real results and risks, then prioritize fixes by measured failure categories.

> [!question]- Q57. An executive wants "100% accurate answers." Respond.
> Reframe with measurable targets on real questions (accuracy, abstention, citation validity, zero permission violations), show current metrics and failure modes, design human review/escalation for high-risk cases, and commit to a monitoring-and-improvement loop rather than a guarantee.

> [!question]- Q58. The customer requires on-prem/air-gapped. What changes?
> Open-weight LLM and embedder/reranker served locally (vLLM, etc.), self-hosted vector DB/pgvector, offline parsing/OCR, internal observability, hardware sizing, and a plan for model updates and evaluation without external APIs.

> [!question]- Q59. Pick the stack for a 5M-chunk, 200-tenant, SOC2 customer already on AWS Postgres.
> Start with pgvector (partitioned/filtered by tenant) or the AWS-native option the customer approves, hybrid with Postgres FTS/BM25 extension or OpenSearch, a managed reranker or self-hosted model, ACL metadata sync, per-tenant leakage tests, observability, and a re-index plan; validate performance with their filters before committing.

> [!question]- Q60. How do you know the system is getting better week over week?
> A versioned golden set run in CI/nightly, dashboards for retrieval/answer metrics by slice, sampled online judge scores calibrated to humans, user feedback conversion into new test cases, and release notes tying changes to metric deltas and cost.

Next → [[11-Resources-and-Practice-Links]]
