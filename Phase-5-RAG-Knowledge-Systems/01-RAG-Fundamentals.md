---
tags: [phase-5, rag, fundamentals]
---
# 01 — RAG Fundamentals

## 1. The problem RAG solves
LLMs have: (a) a **knowledge cutoff**, (b) **no access to private data**, (c) **hallucination** on long-tail facts, (d) **no provenance** (can't say where an answer came from). RAG addresses these by retrieving relevant passages at query time and instructing the model to answer *from them*, with citations.
- Origin: Lewis et al., 2020 (*Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*). Today "RAG" broadly means any system that retrieves external knowledge into the prompt.

## 2. The canonical pipeline
```
OFFLINE (indexing)                                   ONLINE (query)
documents → parse → clean → chunk → (enrich) →       user query → (rewrite/route) → embed query
embed → store in index (vectors + text + metadata)   → retrieve top-k (vector ± keyword, with filters)
                                                      → rerank → assemble context → LLM generates
                                                      grounded answer + citations → validate → return
```
Each box is a **tunable, testable component** — RAG quality is the *product* of its stages. Typical failure is **not** the LLM; it's upstream (bad parsing, bad chunks, retrieval misses).

## 3. RAG vs long context vs fine-tuning
| Approach | Best for | Weaknesses |
|---|---|---|
| **RAG** | large/changing corpora, per-user permissions, citations, freshness, cost control | retrieval can miss; pipeline complexity; multi-hop/global questions hard |
| **Long-context stuffing** | one bounded doc or small corpus; quick prototypes; whole-doc reasoning (summarize, compare) | cost/latency per call; context rot & lost-in-the-middle; no per-user ACLs; can't scale to millions of docs. Prompt caching makes repeated Q&A on one fixed doc cheaper |
| **Fine-tuning** | style/format/behavior, domain terminology, latency/cost at scale, tool-use habits | poor for *facts* (hard to update, no citations, can hallucinate); needs data & eval |
| **Combine** | most real systems: RAG for facts + (optional) fine-tune for behavior + long-context for assembled evidence |
Heuristic: **facts → retrieval; behavior → prompts/fine-tuning; tiny corpus → just stuff it (and cache it).**
Rule of thumb for corpus size: if all relevant material fits comfortably in context *and* you ask many questions, try long-context + caching first; if the corpus is large, changes often, is access-controlled, or needs auditable citations → RAG.

## 4. Anatomy of failure modes (the "RAG failure map")
| Stage | Failure | Symptom |
|---|---|---|
| **Parsing** | tables flattened, columns interleaved, scans unread, headers/footers pollute | answer misses table data; garbage chunks |
| **Chunking** | chunk cuts the answer in half; too large (diluted) / too small (no context) | "close but wrong" retrievals; missing definitions |
| **Embedding** | domain/jargon mismatch; query–doc asymmetry; language mismatch | semantically relevant doc not retrieved |
| **Indexing** | stale index; missing docs; wrong metadata | outdated or absent answers |
| **Retrieval** | pure-vector misses exact terms (IDs, SKUs, error codes); top-k too small; filters wrong | known-answer queries fail |
| **Ranking** | relevant chunk at rank 15, context only takes 5 | answer exists in index but not used |
| **Context assembly** | duplicates, irrelevant noise, poor ordering, no source labels | distractors → wrong answer / lost in middle |
| **Generation** | ignores context, mixes in parametric memory, over-trusts bad passage, no abstention | hallucination despite right context |
| **Citations** | wrong/fabricated source IDs | untrustworthy provenance |
| **Security** | retrieved doc contains instructions (injection); user sees docs they shouldn't | leaks/hijacks |
| **Eval gap** | no golden set → can't tell if changes help | regressions ship |
**Debug in order:** *Was the answer in the corpus? → was it parsed? → was it chunked intact? → was it retrieved (top-k)? → ranked high? → in the prompt? → used correctly?* Measure each hop (Note 07).

## 5. Naive RAG (the baseline you must beat)
```python
chunks = split(doc, 500 tokens)                 # fixed-size
index = embed(chunks)                           # one embedding model
top = knn(embed(query), index, k=5)             # cosine
answer = llm(f"Answer using only:\n{top}\n\nQ: {query}")
```
It works on demos and fails on production queries (exact identifiers, tables, multi-hop, ambiguity, permissions, long docs). **Advanced RAG** = better parsing/chunking + hybrid retrieval + reranking + query transformation + evaluation. **Modular/agentic RAG** = retrieval as a tool in a loop (Note 06).

## 6. Types of questions (match architecture to query type)
| Query type | Example | What works |
|---|---|---|
| **Factoid/lookup** | "What's the refund window?" | dense/hybrid + small chunks |
| **Exact-match/identifier** | "Error E-4021", SKU, clause "7.3(b)" | BM25/keyword + metadata filter |
| **Multi-passage synthesis** | "Summarize our SSO requirements" | larger k, rerank, map-reduce |
| **Multi-hop** | "Which vendor supplies the part used in the product that failed in Q3?" | query decomposition, iterative/agentic retrieval, graph |
| **Aggregation/analytical** | "How many tickets last month by category?" | **SQL/structured tools, not RAG** |
| **Global/thematic** | "What are the main themes across all interviews?" | map-reduce summaries, hierarchical (RAPTOR), GraphRAG community summaries |
| **Comparison** | "Compare policy A and B" | retrieve both separately (per-entity retrieval) |
| **Conversational follow-up** | "And for contractors?" | query rewriting using chat history |
| **Time-sensitive** | "latest pricing" | recency filters/boosting; freshness pipeline |
| **Unanswerable** | question outside corpus | abstention behavior must be tested |
Real systems **route** queries to the right strategy.

## 7. When NOT to use (plain) RAG
- Questions are **structured/numeric** → text-to-SQL or API tools.
- Corpus is tiny and static → long-context + caching.
- You need **reasoning over entire documents** (contract-wide consistency) → long-context or map-reduce, maybe with retrieval for navigation.
- Need **exact, deterministic lookups** (by ID) → plain database/search.
- Latency budget can't afford multi-stage retrieval → precomputed answers/FAQs, caching.

## 8. Success metrics (define before building)
Business: deflection rate, time saved, answer acceptance, escalation rate, CSAT. System: **recall@k** of relevant evidence, answer correctness, faithfulness, citation accuracy, abstention accuracy, p95 latency, cost/query, freshness lag, permission-violation rate (must be 0). Tie to a **golden set** built with domain experts.

## 9. Vocabulary cheat-sheet
Corpus · document · chunk/passage · embedding · index · ANN · top-k · recall · precision · BM25 · dense/sparse/hybrid · reranker/cross-encoder · query rewriting · HyDE · metadata filter · grounding · faithfulness · citation · abstention · context window · context stuffing · parent–child retrieval · GraphRAG · agentic RAG.

## Self-check
1. When is long-context + caching better than RAG? 2. List the stages where RAG fails and one symptom each. 3. Why is "aggregate last month's tickets by category" not a RAG problem? 4. What does "answer in corpus but not in prompt" tell you to inspect? 5. Why define a golden set before tuning?

Next → [[02-Embeddings]]
