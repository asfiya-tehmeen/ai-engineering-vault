---
tags: [phase-5, embeddings]
---
# 02 — Embeddings

> You met embeddings in [[../Phase-2-ML-Deep-Learning/07-Deep-Architectures]] (word2vec, contrastive learning, InfoNCE). Here they become your retrieval engine.

## 1. What an embedding model does
Maps text (or images/audio) → a **dense vector** (e.g., 384–4096 dims) such that **semantically similar inputs are close**. Query and document vectors live in the same space; retrieval = nearest-neighbor search.
- Modern text embedding models are typically **transformers** (BERT-style encoders or LLM backbones — decoder-only LLMs fine-tuned for embeddings are now top performers) trained with **contrastive learning**: pull (query, relevant passage) pairs together, push negatives apart (InfoNCE with in-batch and **hard negatives**), on large mined/synthetic pair datasets, often with multi-stage training and LLM-generated data.
- **Pooling:** mean-pool token states, CLS token, or last-token (LLM-based). Output usually **L2-normalized**.
- **Asymmetric retrieval:** queries are short, documents long. Many models want **instruction/prefix** text: e.g., `"query: ..."` / `"passage: ..."`, or task instructions. **Using the wrong prefix silently hurts recall** — read the model card.

## 2. Similarity metrics
| Metric | Formula | Notes |
|---|---|---|
| **Cosine** | `a·b/(‖a‖‖b‖)` | scale-invariant; most common |
| **Dot product** | `a·b` | equals cosine if vectors are normalized; faster; some models trained for dot |
| **Euclidean (L2)** | `‖a−b‖` | equivalent ranking to cosine on normalized vectors |
Match the metric the model was trained for; normalize if unsure; index with the same metric you query with.

## 3. Representation families
| Type | How | Strength | Weakness |
|---|---|---|---|
| **Dense** (single vector) | one vector per text | semantic matching, paraphrase, cross-lingual | misses exact tokens/IDs; compression loses detail |
| **Sparse / lexical** (BM25, TF-IDF) | term-weight vectors over vocabulary | exact keywords, rare terms, explainable, no training | vocabulary mismatch ("car" vs "automobile") |
| **Learned sparse** (SPLADE, BGE-M3 sparse) | neural term expansion into sparse vector | best of both for lexical+semantic; inverted-index friendly | needs special indexes |
| **Multi-vector / late interaction** (ColBERT, ColPali for page images) | one vector **per token/patch**; score = Σ max-sim (MaxSim) | fine-grained matching, strong accuracy | storage ×(tokens), heavier infra |
| **Hybrid** | combine dense + sparse (+ multi-vector) | robust | more moving parts |
Some models (e.g., BGE-M3) output dense + sparse + multi-vector from one model.

## 4. Dimensions, Matryoshka, and quantization
- Typical sizes: 384 (small/fast), 768, 1024, 1536, 3072, 4096. **Bigger ≠ always better**; storage and search cost scale with dimension.
- **Matryoshka Representation Learning (MRL):** model trained so that **truncating** the vector (e.g., 1024 → 256) retains most quality; then re-normalize. Cheap storage/latency knob; supported by several current models/APIs (check).
- **Vector quantization:** float32 → float16 → int8 → **binary** (32× smaller; use Hamming distance, rescore top candidates with full vectors). Test recall loss on your data.
- **Memory math:** `N × dim × bytes`. 1M vectors × 1024 dims × 4 B = **4 GB** (fp32), 2 GB (fp16), 1 GB (int8), 128 MB (binary) — *plus* index overhead (HNSW graph adds maybe 20–100%+).

## 5. Choosing an embedding model (the honest process)
**Don't trust a single leaderboard number.** Sources in 2026 conflict on who "leads" (different MTEB sub-leaderboards, dates, and self-reported scores); MTEB averages across many tasks that may not resemble yours; and models can be tuned toward public benchmarks.
**Procedure:**
1. **Shortlist** 3–5 candidates from the **MTEB leaderboard** (filter by *Retrieval* task, your language(s), and model size/license/context length) spanning: a hosted API model (e.g., Voyage, OpenAI, Google Gemini, Cohere), a strong open-weight model (e.g., Qwen3-Embedding, BGE family, E5/GTE, NVIDIA Nemotron embed, Jina), and a small fast baseline.
2. **Build a small golden retrieval set from YOUR corpus** (≥50–100 queries with known relevant chunks; Note 07).
3. **Measure** recall@k / nDCG@k / MRR for each, plus **latency, cost, dimension, max input length, license, data-privacy terms**.
4. Check **slices** (short vs long queries, jargon/IDs, languages, tables).
5. Pick the best *trade-off*, not the top score. Re-evaluate when you change chunking or when new models release (embedding **migrations require re-indexing everything**).
Factors beyond accuracy: **max tokens per input** (chunks must fit), **multilingual** needs, **domain** (code, legal, biomedical, finance have specialized models), **deployment** (API vs self-host vs on-prem), **rate limits/throughput** for bulk indexing, **cost** (index once + query forever), **vendor lock-in** (vectors from different models are incompatible).
> Note on providers: Anthropic's API is a text-generation API; for embeddings people commonly pair Claude with an embeddings provider (e.g., Voyage AI is the one Anthropic has historically pointed to) or an open model — **check current docs** for any first-party embedding offering.

## 6. Using embeddings in code
```python
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer("BAAI/bge-small-en-v1.5")          # small open model; read its card for query prefix rules
docs = ["Refunds are issued within 14 days.", "Our API rate limit is 100 req/min.", "Reset your password from Settings."]
D = model.encode(docs, normalize_embeddings=True, batch_size=64)            # (n, 384)
q = model.encode(["how long do refunds take?"], normalize_embeddings=True)  # some models need a query prefix/instruction
scores = (D @ q.T).ravel()                                                  # cosine because normalized
top = np.argsort(-scores)[:3]
```
- **Batch** embedding calls (APIs accept arrays; respect token limits and rate limits); use async/Batch endpoints for bulk indexing; cache embeddings by content hash to avoid re-embedding unchanged chunks.
- **Query-time latency** includes embedding the query (API round trip ~tens–hundreds of ms) — consider a local small model or caching frequent queries.
- Version your index with `(model_name, model_version, dimension, normalization, chunker_version)` metadata.

## 7. Domain adaptation & fine-tuning embeddings
- When generic models underperform on jargon-heavy corpora: **fine-tune** with (query, positive passage[, hard negative]) pairs using Sentence-Transformers (MultipleNegativesRankingLoss). Data sources: real queries + clicks, **LLM-synthesized queries per chunk** (validate quality!), mined hard negatives.
- Evaluate on a held-out golden set; guard against overfitting; compare against simply adding a **reranker** or **hybrid** (often cheaper wins).
- Alternatives: **adapters/linear transforms on frozen embeddings**, **query-side-only fine-tuning**, **prompt/instruction tuning** of instruction-aware embedders.

## 8. Multilingual & cross-lingual
Use multilingual models (BGE-M3, multilingual E5, Qwen3-Embedding, Gemini/Cohere/Voyage multilingual); evaluate on *each* language; cross-lingual retrieval (query in one language, docs in another) works but is weaker; for mixed corpora consider language-specific indexes or translate-then-embed for weak languages.

## 9. Multimodal embeddings
- **CLIP/SigLIP-style** (image–text in one space) for image search; **ColPali/ColQwen**-style late-interaction over **page images** for visually rich documents (skip OCR/layout parsing); multimodal embedding APIs (Cohere Embed v4, Voyage multimodal, Gemini embeddings, Jina) embed text+images jointly. Useful for slides, scanned forms, charts. Evaluate vs OCR-then-text baseline.

## 10. Failure modes & gotchas
- **Lexical blind spots:** product codes, error IDs, names, numbers → add BM25/hybrid (Note 05).
- **Negation/quantifiers/temporal** ("not covered", "before 2020") often confuse dense retrievers → rerank; filters.
- **Hubness / generic chunks** appearing for everything → dedupe, filter boilerplate, add context.
- **Length bias:** very long chunks dilute meaning; truncated inputs lose content silently (check max tokens!).
- **Query–doc mismatch:** "questions" vs "statements" — HyDE/synthetic-question indexing help.
- **Anisotropy / poor calibration:** raw cosine scores aren't comparable across queries → don't use a global similarity threshold without calibration.
- **Embedding drift when you swap models** → full re-index + re-eval.
- **Privacy:** embeddings can leak information about source text; treat as sensitive data (inversion attacks exist).
- **Evaluation leakage:** public benchmarks may be in training data.

## 11. Embeddings beyond retrieval
Clustering/topic discovery (HDBSCAN/BERTopic), deduplication (near-dup detection), classification with a linear head on frozen embeddings (strong cheap baseline), anomaly detection, recommendations, semantic caching, routing (embed query → nearest intent), data exploration (UMAP — [[../Phase-2-ML-Deep-Learning/03-Classical-ML-Algorithms]]).

## Self-check
1. Why normalize vectors? 2. When do you need a query prefix? 3. What's Matryoshka truncation and what must you do after truncating? 4. Why is a leaderboard score insufficient to choose a model? 5. Compute memory for 20M × 768-d fp16 vectors. 6. Why do embeddings miss "ERR-4021"?

Next → [[03-Vector-Search-and-Databases]]
