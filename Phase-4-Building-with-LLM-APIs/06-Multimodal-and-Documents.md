---
tags: [phase-4, multimodal, documents, vision]
---
# 06 — Multimodal Inputs & Document Pipelines

> Enterprise data is mostly **PDFs, scans, screenshots, spreadsheets, slides, emails and audio**. As an FDE, document intelligence will be one of your most common engagements (invoices, contracts, claims, KYC, reports, support attachments).

## 1. Images (vision)
Send images as **base64** or **URL** content blocks, *before* the text question.
```python
import base64
img = base64.standard_b64encode(open("invoice.png", "rb").read()).decode()
r = client.messages.create(
    model="claude-sonnet-5-5", max_tokens=4000,
    messages=[{"role": "user", "content": [
        {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": img}},
        {"type": "text", "text": "Extract vendor, invoice number, date, line items, and total as JSON."},
    ]}])
```
- **Token cost scales with image size/resolution** (providers resize/tile). Count tokens; downscale when detail isn't needed; crop to the region of interest; send multiple images for multi-page.
- Good at: OCR-free reading of clean documents, charts/diagrams, UI screenshots, handwriting (variable), receipts, product photos.
- Weak at: precise counting, fine spatial reasoning/coordinates, tiny text, dense tables in low-res scans, subtle visual differences → **validate** (sums match, required fields present), use higher resolution/crops, human review for high-stakes.
- Put multiple images **with labels** ("Image 1: front, Image 2: back") in text.
- Privacy: images may contain PII (faces, IDs). Apply redaction/retention policies; check data-retention terms.

## 2. PDFs & documents
Providers accept PDFs natively (text extraction + page images). Anthropic: `document` block (base64, URL, or Files API reference); limits on size/pages (e.g., 32 MB request, ~600 pages — verify); place the document before the question.
```python
pdf_b64 = base64.standard_b64encode(open("contract.pdf", "rb").read()).decode()
r = client.messages.create(
    model="claude-opus-5-5", max_tokens=8000,
    messages=[{"role": "user", "content": [
        {"type": "document", "source": {"type": "base64", "media_type": "application/pdf", "data": pdf_b64},
         "citations": {"enabled": True}},
        {"type": "text", "text": "List termination clauses and notice periods."},
    ]}])
```
- **Citations:** enable so answers carry `cited_text` + location (page numbers for PDF, char ranges for text). Great for auditability — but **citations can't be combined with schema-constrained structured outputs** on Claude (400); do two steps (answer with citations → structure separately) or have the schema include `quote` fields.
- **Files API:** upload once (`file_id`), reference across requests (saves bandwidth, enables caching patterns). Match block type to file MIME type.
- **Token reality:** a 100-page PDF can be 100k+ tokens (text + page images). Use **prompt caching** when asking many questions of the same doc; for very large corpora use **RAG** (Phase 5).
- **Other formats:** DOCX/XLSX/PPTX → convert to text/markdown/PDF first (python-docx, openpyxl, `markitdown`, `docling`, `unstructured`); HTML → clean text; email (EML/MSG) → parse headers + body + attachments; CSV → send a sample + schema, use **code execution** for analysis instead of dumping rows.

## 3. Document-processing pipeline (reference architecture)
```
ingest (upload/email/S3) → classify doc type → parse/OCR (native PDF | docling | cloud OCR) 
→ route to extractor prompt/schema per type → LLM extraction (structured output, citations/quotes)
→ validate (schema + business rules + cross-field checks) → confidence/uncertainty scoring
→ human review queue for low-confidence/failed → write to system of record (idempotent) → audit log + metrics
```
Design points:
- **Classify first, then specialized prompts/schemas** (invoice vs PO vs contract) — better than one mega-prompt.
- **Hybrid parsing:** layout-aware OCR/parsers for tables + LLM for semantics; or send page images to a vision model for tricky layouts. Evaluate both on *your* docs.
- **Chunking long docs** (when beyond context or for cost): by section/page with overlap; map-reduce (extract per chunk → merge/reconcile); keep page references.
- **Validation rules:** totals = sum(lines), dates parse and are plausible, IDs exist in master data, currency codes valid, required fields non-null. Failures → retry with error feedback → human.
- **Confidence:** request evidence quotes; compare two independent extractions (disagreement ⇒ review); logprobs where available; model self-reported confidence is weak alone.
- **Human-in-the-loop UI** with highlighted source regions speeds review and generates labeled data for evals/fine-tuning.
- **Golden set:** 50–200 documents with human-verified fields; track field-level precision/recall, exact-match, and **straight-through-processing rate**; slice by doc type/vendor/scan quality ([[../Phase-2-ML-Deep-Learning/04-Evaluation-and-Metrics]]).
- **Security/compliance:** PII/PHI handling, data residency, retention (check ZDR eligibility of features), encryption, access control, audit trails.

## 4. Audio & video
- **Speech-to-text:** Whisper-class models (open or API), cloud STT (Google, AWS Transcribe, Deepgram, AssemblyAI) with diarization/timestamps. Pipeline: transcribe → LLM summarize/extract. **Native audio-in/out models and realtime voice APIs** (Gemini, OpenAI Realtime, others) support low-latency voice agents; Claude's API focus is text/image/PDF — verify current audio support in docs.
- **TTS:** cloud/neural TTS (ElevenLabs, OpenAI, Google, Azure, open models).
- **Video:** sample frames + transcript; Gemini accepts video natively (long-context); otherwise frame sampling + vision model + audio transcript. Token/cost heavy — sample wisely.
- **Voice agent latency budget:** STT (≤300 ms) + LLM TTFT + TTS first-byte; streaming at every hop; interruption handling (barge-in); VAD.

## 5. Image/file generation
- Image generation via provider models (OpenAI, Gemini, others) or diffusion (Stable Diffusion/Flux). 
- **Code-execution sandbox** can generate files (charts with matplotlib, DOCX/PPTX/XLSX/PDF via preinstalled libraries) and return them via the Files API — useful for "make me a report/chart" features. Always sanitize filenames (`os.path.basename`) and confine output directories.

## 6. Web & live data via server tools
- **Web search / web fetch** tools give models fresh information with citations; control `allowed_domains/blocked_domains`, `max_uses`; budget cost per search; treat results as **untrusted** (prompt injection). Web fetch only retrieves URLs already present in the conversation (e.g., from search results or provided by the user).
- **Code execution** for data analysis: upload CSV (Files API) → model writes/executes Python → returns stats and chart files. Container reuse persists state across calls.
- Compare with building your own search/RAG (Phase 5) when you need private data, control over ranking, or compliance.

## 7. Practical evaluation for multimodal
Build a test set stratified by **document quality** (clean, scanned, skewed, low-res, handwriting), **layout** (tables, multi-column, forms), **language**, and **length**. Report accuracy per field and per stratum; examine failure galleries. Compare: native PDF vs page images; model tiers; resolution; with/without OCR pre-pass; with/without cropping.

## 8. Common pitfalls
Sending 300-DPI full-page images when 100-DPI suffices (cost/latency) · trusting extracted numbers without arithmetic checks · mixing citations with schema outputs · forgetting that PDF pages count as images *and* text tokens · not caching repeated doc Q&A · stripping needed context by over-cropping · ignoring rotation/orientation · no human review path · logging raw PII.

## Self-check
1. Why put documents before the question? 2. How do you validate an extracted invoice beyond the schema? 3. When use RAG instead of stuffing a PDF? 4. Why can't citations be combined with schema-constrained output on Claude, and what's the workaround? 5. How do you cut image token cost? 6. Outline a human-in-the-loop document pipeline.

Next → [[07-Multi-Provider-and-App-Patterns]]
