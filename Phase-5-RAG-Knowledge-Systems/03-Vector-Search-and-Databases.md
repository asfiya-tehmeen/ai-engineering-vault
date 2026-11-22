---
tags: [phase-5, vector-search, databases, ann]
---
# 03 — Vector Search & Vector Databases

## 1. The nearest-neighbor problem
Given a query vector `q` and `N` stored vectors, find the top-k most similar. 
- **Exact (brute force / "flat"):** compute all `N` similarities — O(N·d). Perfectly accurate; fine up to ~10⁵–10⁶ vectors (vectorized NumPy/FAISS-Flat/GPU). **Start here** — it's the ground truth for measuring ANN recall.
- **Approximate nearest neighbor (ANN):** trade a little recall for orders-of-magnitude speed at large N. **Recall@k vs exact** is the key quality metric of an index.

## 2. ANN index families
| Family | Idea | Pros | Cons |
|---|---|---|---|
| **HNSW** (Hierarchical Navigable Small World) | multi-layer proximity graph; greedy descent from sparse top layers to dense bottom layer | excellent recall/latency; incremental inserts; default in most DBs | **memory-heavy** (graph + vectors in RAM); slower builds; deletes/updates messy |
| **IVF** (inverted file) | k-means partitions ("cells"); search `nprobe` nearest cells | smaller memory; fast build; good with PQ | needs training; recall depends on `nprobe`; updates drift |
| **PQ / OPQ** (product quantization) | compress vectors into short codes | 8–64× memory reduction | recall loss; rescoring needed |
| **IVF-PQ, HNSW+PQ/SQ** | combos | large-scale | tuning complexity |
| **DiskANN / Vamana, SPANN** | graph/cluster on SSD | billion-scale at lower RAM | more engineering; latency higher than RAM |
| **ScaNN** (Google) | anisotropic quantization | strong speed–recall | library-centric |
| **LSH** | hashing | simple; theoretical guarantees | usually outperformed |
| **Binary quantization + rescoring** | Hamming search then full-precision rerank | huge speed/memory | model-dependent quality |
**HNSW knobs:** `M` (graph degree; ↑ recall & memory), `ef_construction` (build quality), **`ef_search`** (query-time breadth; ↑ recall, ↑ latency — your main runtime dial). pgvector defaults: `m=16`, `ef_construction=64`, `hnsw.ef_search=40`.
**IVF knobs:** `nlist` (cells), `nprobe` (cells searched).
Benchmarks: ann-benchmarks.com, VectorDBBench — but **test with your vectors, filters, and update patterns.**

## 3. Filtering (the hard part in production)
Real queries include **metadata filters**: `tenant_id = X AND doc_type='policy' AND date > …` and **ACLs**.
- **Post-filtering:** ANN top-k then filter → may return < k results or none when filters are selective.
- **Pre-filtering:** filter then search → exact but slow without help.
- **Filtered/integrated ANN** (Qdrant, Weaviate, Pinecone, pgvector's iterative scans, OpenSearch efficient filters): traversal respects filters; behavior under *highly selective* filters varies — **benchmark your selectivity**.
- **Partition by tenant** (namespaces/collections/partitions) for isolation + speed; many small tenants favors object-storage-backed designs.
- Index the metadata columns you filter on (payload/secondary indexes).
- Always **test: no results leak across tenants/ACLs.**

## 4. Vector database landscape (verify live; sources conflict on benchmarks)
| Option | Type | Notes (from 2026 sources — treat as orientation) |
|---|---|---|
| **pgvector** (+ **pgvectorscale**) | Postgres extension | **Default starting point if you already run Postgres**: vectors beside relational rows, SQL joins/filters, transactions, backups, familiar ops. HNSW & IVFFlat; `halfvec`, `bit`, sparse types in recent versions. Scale limits and tuning needs rise at tens–hundreds of millions of vectors / heavy filtering |
| **Qdrant** | open-source dedicated (Rust) | strong filtering, quantization options, hybrid/sparse support, self-host or cloud |
| **Weaviate** | open-source dedicated | built-in hybrid (BM25+vector), modules, multi-tenancy |
| **Milvus / Zilliz** | open-source distributed | very large scale (billions); heavier ops (multiple components) |
| **Pinecone** | managed SaaS | serverless/managed convenience; vendor lock-in/cost considerations |
| **Chroma** | embedded/dev-friendly | great for prototyping/local; evaluate carefully for prod scale |
| **LanceDB** | embedded/serverless columnar (Lance format) | local-first, object-storage friendly, multimodal |
| **turbopuffer** | object-storage-backed serverless | many tenants/namespaces, cost-efficient for cold data |
| **Elasticsearch / OpenSearch / Vespa / Redis / MongoDB Atlas / Cassandra / SQL Server / Oracle** | existing platforms adding vector search | attractive if the customer already operates them (and need hybrid/lexical features) |
| **FAISS / hnswlib / ScaNN / USearch** | libraries | embed in your app; you build persistence/filters |
**Decision guide:** (1) Do you even need a dedicated vector DB? Start with **pgvector** or the customer's existing search/DB. (2) Need scale/ops simplicity/tenancy beyond that → managed (Pinecone/Qdrant Cloud/Weaviate Cloud/turbopuffer) or self-host Qdrant/Milvus. (3) Prototype → Chroma/LanceDB/FAISS. (4) Enterprise constraints (VPC, on-prem, residency, existing vendor approvals) often decide it for you — a key FDE reality.
Evaluate on: recall at target latency **with your filters**, indexing throughput, update/delete behavior, hybrid support, multi-tenancy, backup/DR, security (RBAC, encryption), ops burden, cost model, ecosystem/SDKs, vendor risk.

## 5. pgvector in practice
```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE chunks (
  id bigserial PRIMARY KEY,
  doc_id text NOT NULL,
  tenant_id text NOT NULL,
  content text NOT NULL,
  meta jsonb DEFAULT '{}',
  embedding vector(1024),
  tsv tsvector GENERATED ALWAYS AS (to_tsvector('english', content)) STORED
);
CREATE INDEX ON chunks USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);
CREATE INDEX ON chunks USING gin (tsv);                       -- lexical search
CREATE INDEX ON chunks (tenant_id, doc_id);

-- vector top-k with filter
SET hnsw.ef_search = 100;
SELECT id, content, 1 - (embedding <=> $1) AS cos_sim
FROM chunks WHERE tenant_id = $2
ORDER BY embedding <=> $1 LIMIT 20;                            -- <=> cosine distance, <-> L2, <#> negative inner product

-- keyword top-k
SELECT id, ts_rank_cd(tsv, plainto_tsquery('english', $1)) AS rank
FROM chunks WHERE tsv @@ plainto_tsquery('english', $1) AND tenant_id = $2
ORDER BY rank DESC LIMIT 20;
```
Tips: build the index **after** bulk loading; match the operator class to the distance you query with; raise `ef_search` when filters make results sparse (and use iterative scan options in recent pgvector versions where available); `EXPLAIN ANALYZE` to confirm the index is used; consider `halfvec` to halve memory; partition large tables by tenant; keep `maintenance_work_mem` high for index builds; measure recall vs exact on a sample; mind that Postgres full-text `ts_rank` is not true BM25 (extensions like ParadeDB/pg_search provide BM25).

## 6. Sizing & cost math
- **Storage:** `N_chunks × dim × bytes` + text + metadata + index overhead (HNSW often adds substantial memory; plan for 1.5–2× raw vectors).
- **Example:** 5M chunks × 1024-d fp16 = ~10 GB raw vectors → plan ~20 GB RAM for HNSW + headroom. 100M chunks → consider quantization/DiskANN/sharding.
- **Chunks per document:** a 50-page PDF ≈ 100–300 chunks at ~300–500 tokens each. 100k such docs ≈ 10–30M chunks.
- **Embedding cost** (one-time + deltas): `total_tokens × $/MTok`. Bulk indexing via Batch/async; cache by content hash.
- **Query cost:** embed the query + ANN search + rerank (per-candidate cost!) + LLM tokens (usually the biggest line item).
- **Latency budget example:** query embed 50 ms + ANN 20–50 ms + BM25 20 ms + rerank 100–300 ms + LLM TTFT 500 ms+.

## 7. Data operations
- **Upserts/deletes/updates:** HNSW handles inserts well; deletes/updates can leave tombstones → periodic reindex/vacuum. Design **idempotent ingestion keyed by `(doc_id, chunk_hash)`**; when a doc changes, delete old chunks then insert new ones (atomically).
- **Versioning:** store `embedding_model`, `chunker_version`, `ingested_at`; support **blue/green re-index** (build a new index, shadow-eval, swap).
- **Backups & DR**, **encryption at rest/in transit**, **RBAC**, **audit logs**.
- **Observability:** query latency by percentile, recall probes (periodic exact-vs-ANN checks), index size, ingestion lag, error rates.
- **Multi-region/residency:** replicate or shard by region as required.

## 8. Hands-on from scratch (understand before using a DB)
```python
import numpy as np, time
rng = np.random.default_rng(0)
N, d = 200_000, 384
X = rng.standard_normal((N, d)).astype("float32"); X /= np.linalg.norm(X, axis=1, keepdims=True)
q = rng.standard_normal(d).astype("float32"); q /= np.linalg.norm(q)
t = time.perf_counter(); scores = X @ q; top = np.argpartition(-scores, 10)[:10]; top = top[np.argsort(-scores[top])]
print("brute force ms:", (time.perf_counter()-t)*1e3)          # exact baseline
```
Then try **FAISS** (`IndexFlatIP`, `IndexHNSWFlat`, `IndexIVFPQ`) and measure recall@10 vs exact + latency + memory as you vary `ef_search`/`nprobe`.

## 9. Common mistakes
Choosing a DB by hype · ignoring filter selectivity · no exact-recall baseline · mismatched distance metric/operator class · embedding model changed without re-index · no tenant isolation tests · storing only vectors (not text/metadata/provenance) · unbounded index growth · forgetting deletes (GDPR "right to be forgotten" requires real deletion).

## Self-check
1. Why start with brute force? 2. Explain HNSW in 3 sentences and its main runtime knob. 3. Post- vs pre-filtering trade-offs. 4. Memory for 30M × 768-d int8 vectors? 5. When pgvector, when a dedicated DB? 6. How do you handle document updates and deletions?

Next → [[04-Ingestion-Parsing-and-Chunking]]
