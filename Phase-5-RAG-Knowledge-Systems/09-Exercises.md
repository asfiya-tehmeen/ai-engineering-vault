---
tags: [phase-5, exercises]
---
# 09 — Exercises (Phase 5)

🟢 easy · 🟡 medium · 🔴 hard. **Use a fixed, public corpus first** (e.g., a few hundred pages of PDFs, Wikipedia subset, SEC filings, your favorite open-source project docs) so results are reproducible. Keep experiments in `phase5-exercises/` with a results table (`config → recall@5, MRR, nDCG, answer-correct, latency, cost`) and **bootstrap CIs**. Use small models/local embedders to keep costs near zero.

## A. Fundamentals (Note 01)
- [ ] 🟢 A1. Take 5 questions your favorite LLM answers wrongly about a recent/private document. Paste the doc in context; show the fix. Then explain which RAG failure mode you'd hit at 10,000 docs.
- [ ] 🟢 A2. For each query type (factoid, exact-ID, multi-hop, aggregation, global, comparison, follow-up, unanswerable) write 2 example questions for a corpus of your choice and decide the best strategy (RAG/SQL/long-context/agentic).
- [ ] 🟡 A3. **Long-context vs RAG experiment:** 30 questions over a 150-page PDF. Compare (a) whole-PDF in context with caching, (b) naive RAG, (c) hybrid+rerank RAG. Report accuracy, cost, latency per query, and cost after 100 queries.
- [ ] 🟡 A4. Write a **failure-map debugging checklist** and use it on 10 failing queries from your naive RAG; tag the first broken stage for each; count categories.

## B. Embeddings (Note 02)
- [ ] 🟢 B1. Compute cosine/dot/L2 similarity on normalized vs unnormalized vectors; show when rankings differ.
- [ ] 🟢 B2. Brute-force semantic search over 10k chunks in NumPy; time it; time `argpartition` vs full sort.
- [ ] 🟡 B3. **Embedding bake-off:** 4 models (one API, two open, one small) on a 100-query golden set: recall@5/10, nDCG@10, latency, cost, dims. Include a slice analysis (short vs long queries; IDs vs prose).
- [ ] 🟡 B4. Test the query-prefix/instruction effect: run a model with and without its recommended prefixes; quantify the recall difference.
- [ ] 🟡 B5. Matryoshka/truncation: truncate to 1024/512/256/128 dims (+ renormalize) for a model that supports it; plot recall vs dimension; compute storage savings.
- [ ] 🟡 B6. Quantization: fp32 vs fp16 vs int8 vs binary (+ rescoring top-100 with fp32); measure recall loss and speed.
- [ ] 🟡 B7. Build an embedding-based **near-duplicate detector** and find duplicates in your corpus; show how dedup changes retrieval metrics.
- [ ] 🔴 B8. **Fine-tune an embedder** (MultipleNegativesRankingLoss) on 2–5k (query, passage) pairs (real + LLM-synthesized, filtered); evaluate on held-out real queries; compare against adding a reranker.
- [ ] 🔴 B9. Multilingual: evaluate a multilingual model on queries in 2 languages against docs in another; measure the cross-lingual gap and test translate-then-embed.
- [ ] 🔴 B10. Visualize embeddings with UMAP; cluster with HDBSCAN; label clusters with an LLM; find topics and outliers in your corpus.

## C. Vector search & DBs (Note 03)
- [ ] 🟢 C1. FAISS `IndexFlatIP` baseline vs `IndexHNSWFlat`: measure recall@10 vs exact and latency while sweeping `efSearch`.
- [ ] 🟡 C2. FAISS `IndexIVFFlat`/`IVFPQ`: sweep `nlist`/`nprobe`; plot recall–latency–memory curves.
- [ ] 🟡 C3. Stand up **pgvector** with HNSW; load 500k+ vectors; compare to exact; tune `m`, `ef_construction`, `ef_search`; `EXPLAIN ANALYZE` to confirm index use.
- [ ] 🟡 C4. **Filter selectivity test:** same query with filters selecting 90%, 10%, 1%, 0.1% of rows; measure recall/latency; fix poor cases (higher `ef_search`, iterative scan, partitioning).
- [ ] 🟡 C5. Memory calculator script: given `N, dim, dtype, index type` estimate RAM/disk; validate against real DB stats.
- [ ] 🟡 C6. Implement **idempotent upserts** keyed by content hash and **delete propagation** for removed documents; test with simulated doc updates.
- [ ] 🔴 C7. Compare two vector DBs (e.g., pgvector vs Qdrant or Weaviate) on your data: recall at fixed latency with filters, indexing throughput, update behavior, ops complexity. Write a recommendation memo.
- [ ] 🔴 C8. **Multi-tenancy:** 100 tenants with skewed sizes; implement per-tenant filtering and per-tenant partitioning; run a leakage test proving no cross-tenant results; measure performance for the largest and smallest tenants.
- [ ] 🔴 C9. Blue/green re-index: build a new index with a different embedder, shadow-query both, compare metrics, swap atomically, keep rollback.

## D. Ingestion & chunking (Note 04)
- [ ] 🟢 D1. Parse 20 diverse PDFs with 3 tools (e.g., PyMuPDF, Docling, Unstructured); build a table of failures (tables, columns, headers/footers, scans); pick per-type winners.
- [ ] 🟡 D2. Detect scanned pages automatically (near-empty text) and route them to OCR or a vision model; measure accuracy on 20 scanned pages.
- [ ] 🟡 D3. **Chunking sweep:** fixed-size (256/512/1024 tokens ± overlap), recursive/structure-aware, sentence-window, parent–child. Measure recall@k/nDCG and tokens retrieved per answer.
- [ ] 🟡 D4. Add heading-path/title prefixes to chunks; measure the gain.
- [ ] 🟡 D5. **Table handling:** compare (a) table flattened to text, (b) Markdown table chunks with captions, (c) row-sentences, (d) SQL lookup, on 30 numeric questions.
- [ ] 🟡 D6. Build a metadata schema + ACL fields; implement permission-aware retrieval filters; write tests for 5 users with different groups.
- [ ] 🔴 D7. **Contextual retrieval:** generate chunk-specific context with an LLM (use prompt caching for the document), prepend before embedding and BM25; measure retrieval gain and index-time cost; run with the Batch API.
- [ ] 🔴 D8. Doc2query: generate 3 hypothetical questions per chunk; index them (map back to chunks); compare recall for question-style queries.
- [ ] 🔴 D9. Incremental pipeline: connector → queue → parse → chunk → embed → upsert with retries, dead-letter queue, metrics, and idempotency; simulate crashes mid-run and prove no duplicates/loss.
- [ ] 🔴 D10. Code corpus: AST-based chunking (tree-sitter) vs naive splitting on a repo; evaluate on 40 code-question queries.
- [ ] 🔴 D11. Vision-based parsing: send page images to a multimodal LLM to emit Markdown; compare against OCR+layout parser on tricky pages; measure cost/accuracy.

## E. Retrieval & ranking (Note 05)
- [ ] 🟢 E1. BM25 from scratch (inverted index, IDF, k1, b); verify against `rank_bm25`.
- [ ] 🟢 E2. Implement RRF; test on synthetic rank lists; show robustness to score scale differences.
- [ ] 🟡 E3. **Dense vs BM25 vs hybrid:** table of recall@5/10 and MRR; slice by query type (semantic vs exact-ID).
- [ ] 🟡 E4. Tune weighted fusion `α` vs RRF `k`; show sensitivity.
- [ ] 🟡 E5. Add a **cross-encoder reranker** to top-50 hybrid candidates; plot nDCG@10 vs latency for candidate sizes 10/25/50/100/200.
- [ ] 🟡 E6. Compare rerankers: small open cross-encoder vs larger open model vs API reranker vs LLM-as-reranker (small model); gain/latency/cost table.
- [ ] 🟡 E7. Query rewriting for conversations: 30 multi-turn dialogues; measure retrieval with/without history-aware rewriting.
- [ ] 🟡 E8. Multi-query + RRF; HyDE; compare gains and failure cases.
- [ ] 🟡 E9. Implement MMR and per-document caps; measure diversity vs recall.
- [ ] 🔴 E10. **Router:** small classifier/LLM routes among {no-retrieval, RAG-docs, SQL, web search, clarify}; evaluate routing accuracy and end-to-end cost/quality vs always-RAG.
- [ ] 🔴 E11. Late interaction: try ColBERT (RAGatouille) as retriever and as reranker; compare accuracy/latency/storage vs bi-encoder + cross-encoder.
- [ ] 🔴 E12. Learned sparse (SPLADE or BGE-M3 sparse) vs BM25 in a three-way hybrid; test whether it beats two-way hybrid on your data.
- [ ] 🔴 E13. Fine-tune a reranker on (query, pos, hard-neg) triples; evaluate on held-out real queries.

## F. Generation & advanced RAG (Note 06)
- [ ] 🟢 F1. Write 3 generation prompts (minimal, structured with abstention, quote-first); compare faithfulness/correctness on 50 questions.
- [ ] 🟡 F2. Citation verification: implement quote-matching + entailment check; measure citation precision on 100 answers.
- [ ] 🟡 F3. Abstention calibration: plot answer/abstain decisions vs top reranker score; choose τ; report false-answer vs false-abstain trade-off on answerable + unanswerable sets.
- [ ] 🟡 F4. Context-size ablation: pass k = 1, 3, 5, 10, 20 chunks; measure accuracy, cost, and "lost in the middle" effects (rotate relevant chunk position).
- [ ] 🟡 F5. Conflicting documents (v1 vs v2 policy): add date metadata and instructions; test the model prefers the current version.
- [ ] 🟡 F6. Prompt-injection drill: plant a malicious instruction in a document; test whether the system obeys; implement mitigations (delimiting, instruction hierarchy, tool restrictions) and re-test.
- [ ] 🔴 F7. **Agentic RAG:** implement tools `search(query, filters)`, `fetch_section(doc, section)`, `list_outline(doc)`; loop with step/budget caps; compare to single-shot on 40 multi-hop questions (accuracy, tokens, latency, trajectory sanity).
- [ ] 🔴 F8. **Corrective RAG:** grade retrieved chunks (relevant/ambiguous/irrelevant) → rewrite query or fall back to another source; measure improvement on hard queries.
- [ ] 🔴 F9. **GraphRAG-lite:** LLM-extract entities/relations from 200 docs into a graph (Neo4j or networkx); answer multi-hop questions via graph traversal + vector retrieval; compare against hybrid + decomposition; record indexing cost.
- [ ] 🔴 F10. **RAPTOR-style hierarchy:** cluster chunks → summarize → tree; evaluate on thematic questions vs flat RAG.
- [ ] 🔴 F11. SQL + RAG assistant: route between text-to-SQL (customers/orders table) and policy docs; test mixed questions requiring both.
- [ ] 🔴 F12. Multimodal RAG: page-image retrieval (ColPali-style or multimodal embeddings) + vision LLM vs OCR-text pipeline on 40 chart/table questions.

## G. Evaluation & production (Note 07)
- [ ] 🟢 G1. Create a golden set of 50 real-style queries with labeled relevant docs/pages and reference answers for a corpus you know. Document labeling guidelines.
- [ ] 🟡 G2. Implement recall@k, MRR, nDCG, hit-rate; unit-test with hand-made cases; add bootstrap CIs and paired comparison.
- [ ] 🟡 G3. **LLM judge calibration:** hand-label 40 answers (faithful/correct); compute judge agreement (accuracy/κ); iterate the judge prompt; check position/verbosity bias.
- [ ] 🟡 G4. Demonstrate the **"faithful but wrong"** and **"retrieval missed but parametric memory saved it"** pitfalls with concrete examples.
- [ ] 🟡 G5. Add tracing (Langfuse/Phoenix/OpenTelemetry) to your pipeline; build a dashboard (latency by stage, tokens, retrieval scores, abstention rate).
- [ ] 🟡 G6. CI regression gate: golden set runs on every PR; fail if recall@5 or correctness drops beyond CI; publish a report artifact.
- [ ] 🔴 G7. **Synthetic vs real queries:** generate 200 synthetic questions; compare metrics vs 50 real ones; quantify the optimism bias; build a hybrid test set.
- [ ] 🔴 G8. Permission red-team: attempt cross-user/cross-tenant leakage via prompts, query tricks, and cached answers; fix all leaks; add automated tests.
- [ ] 🔴 G9. Cost model: per-query cost breakdown (embed, retrieval, rerank, LLM in/out, rewrite, judge) and monthly projection at 10k/100k/1M queries; identify the top three savings and validate one.
- [ ] 🔴 G10. Freshness: modify/delete source docs and measure time-to-visible-in-answers; implement and test tombstoning; show GDPR-style deletion across chunks, embeddings, caches, and logs.
- [ ] 🔴 G11. Write an **enterprise rollout plan** (discovery → pilot → hardening) for a fictional bank/hospital: data audit, ACL strategy, golden set with SMEs, success metrics, risks, and exec reporting template.

## H. Mini-capstones (pick 2)
- [ ] 🔴 H1. **Support-KB copilot** with hybrid+rerank, citations, abstention, feedback loop, eval CI.
- [ ] 🔴 H2. **Contract review assistant** (clause retrieval, comparison, risk flags) with page-level citations.
- [ ] 🔴 H3. **Codebase Q&A** with AST chunking, repo navigation tools, and answer verification.
- [ ] 🔴 H4. **Research assistant** over arXiv PDFs with multi-hop/agentic retrieval and a bibliography.

## Cadence
3–4 exercises/week; keep a lab notebook of experiments (hypothesis → result → decision) in the vault; review [[10-QA-Interview-Questions]] on Sundays.
