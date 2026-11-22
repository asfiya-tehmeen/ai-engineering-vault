---
tags: [phase-5, ingestion, parsing, chunking]
---
# 04 — Ingestion: Parsing, Cleaning & Chunking

> "Garbage in, garbage out" is most true here. In real engagements **60%+ of RAG quality problems originate in ingestion**: bad parsing and bad chunks. This is unglamorous, high-leverage work — and exactly what FDEs spend time on.

## 1. Sources you'll meet
PDFs (born-digital & scanned), Word/PowerPoint/Excel, HTML/wikis (Confluence, SharePoint, Notion), Google Drive, email, Slack/Teams, tickets (Zendesk/Jira/ServiceNow), CRM records, code repos, databases, transcripts (audio/video), images. Each has its own **connector**, **auth**, **rate limits**, **permissions model**, and **change-detection** story.
Connector tools: LlamaHub/LlamaIndex readers, LangChain loaders, Unstructured, Airbyte, Fivetran, Glean-style enterprise search connectors, custom APIs.

## 2. Parsing (extract structure, not just text)
| Content | Approach |
|---|---|
| **Born-digital PDF** | layout-aware extraction: PyMuPDF/`pdfplumber` (text + coordinates), **Docling**, Unstructured, `marker`, MarkItDown; output **Markdown/HTML with headings, lists, tables** |
| **Scanned PDF/images** | OCR (Tesseract, PaddleOCR, cloud: Textract/Document AI/Azure Document Intelligence) or **vision-LLM** reading page images; or **ColPali-style** page-image embeddings |
| **Tables** | preserve as Markdown/HTML/CSV rows with caption & header; for numeric Q&A prefer **structured storage + SQL** and keep a text summary for retrieval |
| **Slides** | per-slide chunk + speaker notes + slide title; images may need captions |
| **Spreadsheets** | don't flatten blindly: per-sheet schema + sample rows → text-to-SQL/pandas tools |
| **HTML/wiki** | strip nav/boilerplate; keep heading hierarchy, breadcrumbs, canonical URL, last-modified |
| **Email threads** | de-quote replies, keep headers (from/to/date/subject), thread ID, attachments separately |
| **Code** | AST/function-level chunks (tree-sitter), include file path/imports/docstrings |
| **Transcripts** | speaker labels, timestamps, topic segmentation |
| **Charts/diagrams** | caption via vision model; store alt text; link to figure |
**Quality checks:** randomly sample 50 parsed pages and *read them*; compare against the page image; log parse failures; compute % of pages with near-empty text (scans!). Use a **parsing eval set** (pages with known tricky structure).
**Vision-LLM parsing** (send page images to a multimodal model → Markdown/JSON) often beats classic OCR on complex layouts but costs more and can hallucinate small text; combine with validation and sampling ([[../Phase-4-Building-with-LLM-APIs/06-Multimodal-and-Documents]]).

## 3. Cleaning & normalization
- Remove repeated headers/footers/page numbers/watermarks; fix hyphenation across lines; normalize whitespace/Unicode; handle multi-column order; drop boilerplate (cookie banners, nav).
- **Deduplicate**: exact (hash) and near-duplicate (MinHash/SimHash/embedding similarity); keep the canonical/most recent version; duplicates waste top-k slots.
- **Language detection**, **PII detection/redaction** if policy requires (Presidio), **classification of doc type & sensitivity**.
- Resolve **versions/superseded docs** (policy v3 replaces v2) — store `effective_date`, `status`; filter or boost accordingly.

## 4. Metadata (cheap, powerful)
Per chunk store: `doc_id`, `source_url/path`, `title`, `section_path` (H1>H2>H3), `page`, `chunk_index`, `doc_type`, `author`, `created/modified/effective_date`, `language`, `tenant/department`, **ACL (allowed groups/users)**, `version/status`, `chunker_version`, `embedding_model`, content hash. Uses: **filters**, **boosting recency**, **citations**, **permission enforcement**, **debugging**, **incremental updates**.

## 5. Chunking: the core design decision
**Why chunk?** Embedding models have token limits and single vectors blur multiple topics; LLM context is limited and noisy; retrieval granularity should match question granularity.
**Trade-off:** *small chunks* → precise matches but lose context ("it", "the above"); *large chunks* → context-rich but diluted embeddings, more noise, higher token cost.

### Strategies
| Strategy | How | Use |
|---|---|---|
| **Fixed-size (tokens/chars) + overlap** | e.g., 300–500 tokens, 10–20% overlap | baseline; fast |
| **Recursive/structure-aware** | split by headings → paragraphs → sentences until size fits (LangChain `RecursiveCharacterTextSplitter`, Markdown header splitters) | **strong default** |
| **Semantic chunking** | split where embedding similarity between adjacent sentences drops | can help messy prose; costlier; not always better than structure-aware — test |
| **Document-structure chunking** | one chunk per section/clause/FAQ/slide/function/ticket/message | legal/policy/API docs/code/support — **respect natural units** |
| **Sentence-window** | embed sentences, return surrounding window | precise match + context |
| **Parent–child (small-to-big)** | embed small child chunks; at retrieval return the larger **parent** section | precision + context — very effective |
| **Hierarchical/RAPTOR** | recursively summarize clusters → tree; retrieve at multiple levels | global/thematic questions |
| **Late chunking** | embed the full long doc with a long-context embedder, then pool token embeddings per chunk → chunk vectors retain document context | needs long-context embedding models |
| **Proposition/atomic-fact chunking** | LLM rewrites text as standalone facts | high precision; expensive; risk of paraphrase drift |
| **LLM-based chunking** | LLM decides boundaries | costly; evaluate |
| **Agentic/dynamic** | retrieval agent navigates doc structure on demand (outline → section) | big/complex docs |
**Practical defaults to start:** structure-aware recursive splitting at ~300–600 tokens with ~10% overlap, **prepend the heading path to every chunk**, keep tables whole (or table-row chunks with header), then **measure** alternatives on your golden set. Chunk-size sweeps (e.g., 128/256/512/1024) are a standard experiment; optimum is corpus- and query-specific.

### Contextual enrichment (big wins)
- **Contextual Retrieval (Anthropic, 2024):** have an LLM write a short, chunk-specific context ("This chunk is from ACME's Q2 2023 10-K, section on revenue…") and **prepend it to the chunk before embedding *and* BM25 indexing**. Anthropic reported large reductions in failed retrievals (and more when combined with hybrid search + reranking). With **prompt caching** the whole document is cached while generating context for each chunk, making it affordable. Validate on your data.
- **Title/section-path prefixing** (cheap version of the above).
- **Hypothetical questions per chunk (doc2query):** LLM generates questions each chunk answers; index those (map back to chunk) → fixes question–statement mismatch.
- **Chunk summaries / keywords / entities** as extra searchable fields; **multi-vector per chunk** (content + summary + questions).
- Enrichment is an **index-time cost** (LLM calls × chunks) — budget and batch it (Batch API).

### Special cases
- **Tables:** chunk = table + caption + header repeated per slice; or convert rows to sentences ("Product X, Q3 revenue: $4.2M"); numeric Q&A → SQL.
- **Code:** by function/class with file path and signature; embed docstrings + code; consider code-specific embedders.
- **Conversations/tickets:** chunk by thread or by message window with participants/timestamps; keep resolution notes.
- **Long legal/policy docs:** clause-level chunks with `section_number`; parent = article; cross-reference links.
- **Q&A/FAQ:** question as the retrieval key, answer as payload.
- **Multilingual:** chunk per language; store `language`.

## 6. Incremental updates & freshness
- **Change detection:** webhooks/CDC/modified-time polling/hash compare. Re-parse only changed docs.
- **Idempotent upserts:** key = `(doc_id, chunk_ordinal or chunk_hash)`; delete orphans.
- **Tombstones & deletions** (deleted/unpublished docs and revoked access must disappear *quickly*).
- **Freshness SLAs** (e.g., "policy changes searchable within 5 min") → streaming pipeline vs nightly batch; show "last updated" in citations.
- **Backfills & re-index** when changing parser/chunker/embedder: run offline, shadow-evaluate, swap atomically (blue/green).

## 7. Permissions at ingestion (enterprise-critical)
- Capture the **source system's ACLs** per document (groups, users, link-sharing) and sync changes (permission changes without content changes!).
- At query time, **filter retrieval by the requesting user's identity/groups** — *never* retrieve then rely on the LLM to "not mention" restricted content.
- Test with adversarial users; log access; handle group membership changes, nested groups, and inheritance; default-deny when unsure.
- Consider per-tenant indexes for strong isolation.

## 8. Pipeline engineering
Queue-based, idempotent, observable pipeline: `fetch → parse → clean → chunk → enrich → embed → upsert`. Orchestrate with Airflow/Dagster/Prefect/Temporal; retries & dead-letter queues; per-stage metrics (docs/min, failures, tokens, cost); store **raw originals + parsed artifacts** (re-run without re-fetch); config/version everything; **sample-based QA dashboards** (parse quality, chunk-length histograms, duplicate rate).

## 9. Chunking experiment template
1. Fix a golden set (queries → relevant doc/passage IDs).
2. For each config (chunker × size × overlap × enrichment), build an index.
3. Measure recall@5/10, MRR, nDCG; plus avg tokens retrieved; plus end-to-end answer correctness on a subset.
4. Inspect failures by hand; choose per-corpus config; document.

## Self-check
1. Name 5 parsing problems that silently wreck retrieval. 2. Why prepend heading paths/contextual text to chunks? 3. Explain parent–child retrieval. 4. How do you handle a deleted/revoked document? 5. Why shouldn't the LLM enforce permissions? 6. How would you chunk a 300-page contract vs a Slack export vs a Python repo?

Next → [[05-Retrieval-and-Ranking]]
