---
tags: [phase-5, retrieval, reranking, hybrid-search]
---
# 05 — Retrieval & Ranking

> The consistent finding across 2026 practitioner write-ups: **hybrid retrieval (dense + lexical) fused with RRF, then a cross-encoder reranker on the top candidates, is the production default.** Gains vary by corpus (some benchmarks show rerankers barely beating naive RAG) — so **measure on your own golden set**.

## 1. Lexical retrieval: BM25
- **TF-IDF → BM25**: scores documents by term frequency (saturating), inverse document frequency (rare terms matter), and length normalization. Parameters `k1≈1.2–2.0`, `b≈0.75`.
- Strengths: exact matches (IDs, names, error codes, rare jargon), no training, explainable, fast inverted index. Weaknesses: no synonyms/paraphrase, language-specific tokenization/stemming needed.
- Implementations: Elasticsearch/OpenSearch/Lucene, Vespa, Tantivy, `rank_bm25` (in-memory Python), Postgres FTS (`tsvector`; **not** exactly BM25 — extensions like ParadeDB provide it), SQLite FTS5, Qdrant/Weaviate sparse/BM25 support.
- Tokenization matters: handle `snake_case`, `camelCase`, hyphenated IDs, numbers, code identifiers; consider n-grams for typos/partial IDs.

## 2. Dense retrieval
Embed query → ANN top-k ([[03-Vector-Search-and-Databases]]). Good semantic recall; weaker on exact tokens, negation, numerics.

## 3. Hybrid retrieval
Run **both**, then fuse:
- **Reciprocal Rank Fusion (RRF):** `score(d) = Σ_r 1/(k + rank_r(d))`, `k≈60`. Rank-based → no need to normalize incomparable scores; robust default.
- **Weighted score fusion:** normalize scores (min–max/z-score) then `α·dense + (1−α)·sparse`; tune `α` on the golden set; sensitive to score distributions.
- **Learned sparse (SPLADE/BGE-M3 sparse)** + dense in one DB; **three-way hybrid** (dense + BM25 + learned sparse) followed by a strong reranker was reported among the most accurate configurations in some recent benchmarks — verify for your data.
```python
def rrf(rank_lists, k=60):
    scores = {}
    for ranks in rank_lists:                       # each: [doc_id, ...] best→worst
        for r, d in enumerate(ranks, start=1):
            scores[d] = scores.get(d, 0.0) + 1.0 / (k + r)
    return sorted(scores, key=scores.get, reverse=True)
fused = rrf([dense_ids[:50], bm25_ids[:50]])[:50]
```
Typical funnel: **retrieve 50–200 candidates cheaply → rerank to top 5–20 → pass 3–10 chunks to the LLM.**

## 4. Reranking
First-stage retrievers (bi-encoders) embed query and doc *separately* → fast but coarse. **Cross-encoders** read `(query, passage)` **together** with full attention → much more accurate relevance scores, but O(candidates) model calls per query.
- **Open models:** BGE-reranker family, Qwen3-Reranker, Jina reranker, `cross-encoder/ms-marco-*` (Sentence-Transformers `CrossEncoder`), mixedbread. **APIs:** Cohere Rerank, Voyage rerank, Jina, others. **LLM-as-reranker** (listwise/pointwise prompting a general LLM) — flexible and strong but slower/costlier; can use a small fast model.
- **Late-interaction (ColBERT/ColBERTv2/PLAID, RAGatouille, ColPali for images)**: precompute per-token vectors for docs; score by MaxSim; accuracy near cross-encoders with better speed; storage-heavy. Used as first-stage *or* reranker; treat "ColBERT reranker on top of three-way hybrid is best" as a hypothesis to test, not a fact.
```python
from sentence_transformers import CrossEncoder
ce = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")           # small & fast; swap for a stronger reranker
pairs = [(query, c["text"]) for c in candidates]
scores = ce.predict(pairs)
reranked = [c for _, c in sorted(zip(scores, candidates), key=lambda x: -x[0])][:8]
```
- **Cost/latency:** reranking 100 candidates of 300 tokens ≈ 30k tokens through the model; GPU or API required for sub-300 ms. Tune **candidate count** vs latency.
- **Calibration:** reranker scores support **thresholding** ("if top score < τ → abstain/ask clarification") — tune τ on labeled data.
- **Fine-tune rerankers** on your domain (query, positive, hard negatives) — often higher ROI than fine-tuning the embedder.
- Evidence is mixed that rerankers always beat well-tuned hybrid — **A/B on golden set** (nDCG@k, recall@k, end-to-end answer accuracy).

## 5. Query transformation (fix the query, not just the index)
| Technique | What | When |
|---|---|---|
| **Query rewriting / contextualization** | resolve pronouns & follow-ups using chat history ("and for contractors?" → full standalone query) | multi-turn chat (essential) |
| **Multi-query / query expansion** | LLM generates 3–5 paraphrases/sub-queries; retrieve each; fuse (RRF) | vague or vocabulary-mismatch queries |
| **HyDE** (Hypothetical Document Embeddings) | LLM writes a *hypothetical answer*; embed that to search | short/abstract queries; sometimes hurts factual precision |
| **Decomposition** | split multi-part/multi-hop question into sub-questions answered sequentially/in parallel | comparisons, multi-hop |
| **Step-back prompting** | ask a more general question first for background context | conceptual queries |
| **Keyword extraction / entity linking** | extract IDs/names for lexical/filter lookup | identifiers & structured fields |
| **Metadata/filter extraction ("self-query")** | LLM converts NL to filters (`year=2023`, `product=X`) | catalogs, policies, support KBs — **validate filters** |
| **Spelling/acronym expansion & domain glossary** | expand "PTO" → "paid time off" | enterprise jargon |
| **Translation** | translate query/doc for cross-lingual | multilingual corpora |
Use **small, fast models** and cache; each transformation adds latency/cost — keep only those that improve golden-set metrics.

## 6. Routing
A **router** (rules, embedding-classifier, or small LLM) picks a strategy per query:
- which **index/collection** (HR vs engineering docs), which **tool** (RAG vs SQL vs API vs web search), **retrieval depth**, whether retrieval is needed at all ("hello", arithmetic), or whether to **ask a clarifying question**.
- Evaluate the router separately (accuracy per route; cost of misrouting). Provide a fallback route.

## 7. Filtering, boosting, recency & permissions
- **Hard filters** (tenant, ACL, doc_type, language, date range).
- **Soft boosts** (recency decay, source authority, doc status "current"), implemented in fusion or reranker features.
- **Dedup & diversity:** remove near-duplicates; **MMR (Maximal Marginal Relevance)** balances relevance and diversity: pick `argmax λ·sim(q,d) − (1−λ)·max sim(d, selected)`.
- **Collapse by document** when many chunks come from one doc; cap per-doc chunks to widen coverage (or intentionally expand neighbors).

## 8. Context assembly (what the LLM actually sees)
- **Neighbor/parent expansion:** after retrieving a small chunk, include adjacent chunks or the parent section ("small-to-big"), within a token budget.
- **Ordering:** put the most relevant evidence **first and last** (lost-in-the-middle), or group by document in reading order for coherence.
- **Format with source labels** the model can cite:
```
<documents>
  <document index="1"><source>HR-Policy-2025.pdf · §4.2 · p.12</source><content>…</content></document>
  <document index="2"><source>…</source><content>…</content></document>
</documents>
```
- **Budget:** set a max context tokens; choose `k` adaptively (stop when scores drop below threshold). More context ≠ better — distractors reduce accuracy; **precision of the top few** matters.
- **Compression:** extract only relevant sentences (LLM or extractive "contextual compression") or summarize chunks to save tokens; validate it doesn't drop crucial details.
- **Caching:** stable system prompt/instructions first; retrieved docs vary → cache hits only on prefix. If the same docs recur (FAQ), cache them in a prefix.

## 9. Tuning retrieval systematically
1. Build golden set (Note 07). 2. Baseline: dense only, k=5. 3. Add BM25 hybrid → measure. 4. Increase candidate pool + add reranker → measure. 5. Add query rewriting for multi-turn → measure on conversational subset. 6. Tune chunking/enrichment. 7. Tune `k`, thresholds, boosts. 8. Check latency/cost; choose the Pareto-best config. **One change at a time, with CIs.**
Metrics: **Recall@k** (is the evidence in the candidates), **MRR/nDCG** (is it ranked high), **context precision** (how much of the context is relevant), end-to-end **answer correctness**.

## 10. Failure analysis cheatsheet
| Symptom | Likely fix |
|---|---|
| Exact identifier not found | add BM25/hybrid; keyword filter; tokenization fixes |
| Relevant chunk found at rank 20 | reranker; larger candidate pool; better embedding |
| Follow-up questions fail | query rewriting with history |
| Right doc, wrong section | smaller chunks + parent expansion; heading-path prefix |
| Multi-hop fails | decomposition/agentic loop/graph |
| Outdated answers | recency boost/filter; freshness pipeline; version metadata |
| Same doc dominates | per-doc cap, MMR |
| Answers when it shouldn't | score threshold + abstention instruction + eval set of unanswerables |

## Self-check
1. Explain RRF and why it avoids score normalization. 2. Bi-encoder vs cross-encoder trade-off. 3. When can HyDE hurt? 4. Why rewrite queries in multi-turn chat? 5. Describe the retrieve→rerank→assemble funnel with example numbers. 6. How do you decide if a reranker is worth its latency?

Next → [[06-Generation-and-Advanced-RAG]]
