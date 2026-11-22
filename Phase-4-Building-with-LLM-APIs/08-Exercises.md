---
tags: [phase-4, exercises]
---
# 08 — Exercises (Phase 4)

🟢 easy · 🟡 medium · 🔴 hard. **Cost control:** use a small model (e.g., `claude-haiku-5-5` or another provider's small tier) and a local OpenAI-compatible server (Ollama/vLLM) for iteration; set provider spend limits first; cap `max_tokens` and loop iterations. Put code in `phase4-exercises/`; keep prompts in files; log usage for every call.

## A. API fundamentals (Note 01)
- [ ] 🟢 A1. Make the same request three ways: `curl`, raw `httpx`, and the SDK. Compare request/response shapes; print every response field.
- [ ] 🟢 A2. Write a CLI chat loop with history. Show token usage after each turn and the running cost; demonstrate cost growth per turn.
- [ ] 🟢 A3. Trigger each error on purpose (bad key → 401, bad model → 404, huge `max_tokens`/invalid params → 400) and print the typed exception class.
- [ ] 🟢 A4. Force `stop_reason == "max_tokens"` (tiny limit); write code that detects it and warns.
- [ ] 🟡 A5. Build `cost_meter.py`: wraps calls, records `usage`, computes $ from a price table in YAML, writes JSONL logs, and prints a daily summary by model/feature.
- [ ] 🟡 A6. Compare token counts for the same 5 texts (English, code, Hindi, JSON, emoji) via the provider's `count_tokens` vs `tiktoken`; explain the differences.
- [ ] 🟡 A7. Show that sending `temperature` to a current Claude model fails; find the exact error; then reproduce temperature effects on an open model via Ollama/OpenAI-compatible API (compare T=0, 0.7, 1.5 over 20 samples).
- [ ] 🟡 A8. Try adaptive thinking with `effort` = low/medium/high on 10 reasoning problems: record accuracy, output tokens (incl. thinking), latency, cost. Plot the trade-off.
- [ ] 🟡 A9. Write `LLMClient` with a provider adapter interface and two adapters (Anthropic + an OpenAI-compatible one). Same `complete()` signature.
- [ ] 🔴 A10. Implement client-side conversation persistence in SQLite (conversations, messages with full content blocks). Resume a conversation across processes without losing tool/thinking blocks.
- [ ] 🔴 A11. Use the Models API to list models and capabilities (context window, max output); build a script that picks the cheapest model satisfying given constraints.

## B. Prompt engineering (Note 02)
- [ ] 🟢 B1. Take a vague prompt ("summarize this") and rewrite it five ways adding: audience, purpose, length, format, must/must-not. Run each on 5 docs; judge differences.
- [ ] 🟢 B2. Convert a free-text prompt into XML-tagged sections; test if outputs improve on a long-document QA task.
- [ ] 🟡 B3. Few-shot ablation for a 5-class classification task: 0/1/3/8 examples; shuffled orders; skewed vs balanced examples. Report accuracy ± CI on ≥100 items.
- [ ] 🟡 B4. "Quote-first" vs direct answer on long-document QA; measure hallucination/faithfulness.
- [ ] 🟡 B5. Build a **prompt test harness**: YAML of cases (input, expected/regex/JSON checks), runs N prompt versions, outputs a comparison table (accuracy, cost, latency).
- [ ] 🟡 B6. Take an old "over-engineered" prompt (ALL-CAPS rules, step-by-step scaffolding, role-play) and prune it iteratively; show metrics before/after on a newer model.
- [ ] 🟡 B7. Write a system prompt for a customer-support bot with scope, tone, escalation, uncertainty policy; red-team it with 20 adversarial inputs (jailbreaks, off-topic, injection) and fix failures.
- [ ] 🟡 B8. Prompt chaining: split a "read contract → find risks → draft summary email" mega-prompt into 3 calls; compare quality, cost, latency, debuggability.
- [ ] 🔴 B9. Build a prompt-optimization loop: LLM proposes edits from failure cases → re-test on a train/val split → keep winners. Beware overfitting; report held-out gain.
- [ ] 🔴 B10. Same task, three models (small/mid/flagship) × three effort levels. Build a cost–quality Pareto chart and pick a deployment config.
- [ ] 🔴 B11. Meta-experiment: does "You are a world-class expert" change accuracy on your task? Run 100+ items with CIs and report honestly (often null).

## C. Structured outputs & tools (Note 03)
- [ ] 🟢 C1. Extract contact info from 20 messy emails with Pydantic + `parse`. Report field-level accuracy.
- [ ] 🟢 C2. Add a `null`/"not_found" option and "do not guess" instruction; measure fabrication before vs after.
- [ ] 🟡 C3. Reorder schema fields (`rationale` before `decision` vs after); measure accuracy difference.
- [ ] 🟡 C4. Build a validation+repair loop: on Pydantic error, retry once with the error text; track success rate by attempt; escalate to human queue.
- [ ] 🟡 C5. Write a **hand-rolled tool loop** with 3 tools (calculator, weather stub, order lookup). Include parallel calls, `is_error`, max-iterations, and logging of every call.
- [ ] 🟡 C6. Rewrite C5 using the SDK tool runner; compare code size and control.
- [ ] 🟡 C7. Tool-description A/B: ambiguous vs precise descriptions; measure wrong-tool rate on 50 queries.
- [ ] 🟡 C8. Implement `strict: true` tools and show the model can't emit schema-invalid args; then remove strict and find violations.
- [ ] 🔴 C9. Security drill: build a file-read tool; test path traversal (`../../etc/passwd`) and confine to a root; add SQL tool with parameterization and read-only creds; demonstrate prompt injection via tool output and mitigate (delimiters, permission checks, confirmation).
- [ ] 🔴 C10. Build a **text-to-SQL** assistant on a sample DB (e.g., Chinook): schema in context, read-only role, `sqlglot` validation, row limit, timeout, explain-before-run. Evaluate execution accuracy on 50 gold questions.
- [ ] 🔴 C11. Implement an MCP server exposing 2 tools (stdio) and connect it via the SDK MCP helpers or the provider's MCP connector; compare to native tools.
- [ ] 🔴 C12. Compare structured-output behavior and limits across two providers for the same schema; document differences.

## D. Streaming, async, batch, reliability (Note 04)
- [ ] 🟢 D1. Stream tokens to the terminal; measure TTFT and tokens/sec; show final usage with `get_final_message()`.
- [ ] 🟡 D2. FastAPI SSE endpoint + a minimal HTML/JS client consuming it; add a stop button that cancels the upstream stream.
- [ ] 🟡 D3. Async fan-out of 200 classification calls with `Semaphore(N)`; sweep N (1, 4, 8, 16, 32); plot throughput and 429 rate.
- [ ] 🟡 D4. Add checkpoint/resume to D3 (JSONL results by ID); kill the process mid-run and resume.
- [ ] 🟡 D5. Submit the same 500 requests via the Batch API and compare cost/latency to D3; key results by `custom_id`; handle failures.
- [ ] 🟡 D6. Write a retry wrapper with exponential backoff + jitter honoring `retry-after`; unit-test with mocked 429/500/timeout responses (respx).
- [ ] 🟡 D7. Implement a circuit breaker; simulate provider failures and verify fail-fast + recovery.
- [ ] 🔴 D8. Build a **router**: small model classifies difficulty; easy → small model, hard → flagship. Evaluate cost/quality vs always-flagship and always-small.
- [ ] 🔴 D9. Provider fallback: primary Anthropic → secondary OpenAI-compatible; verify outputs validate against the same schema; chaos-test by blocking the primary.
- [ ] 🔴 D10. Token-aware rate limiter (RPM + TPM budgets) shared across async workers; verify no 429s at saturation.
- [ ] 🔴 D11. Durable workflow: run a 3-step LLM pipeline under Temporal/Inngest (or a Postgres-backed queue) with retries and idempotency; crash workers mid-run and verify exactly-once effects.

## E. Context, caching, cost (Note 05)
- [ ] 🟢 E1. Demonstrate prompt caching: a 10k-token system prompt, 10 requests; print cache write/read tokens and compute savings.
- [ ] 🟡 E2. Break the cache five ways (timestamp in system prompt, shuffled tool order, unsorted JSON, different model, TTL expiry); confirm each via `usage`.
- [ ] 🟡 E3. Multi-turn chat with caching on the growing history; chart cost per turn with and without caching.
- [ ] 🟡 E4. Measure TTFT with vs without cache hit on a 50k-token prefix.
- [ ] 🟡 E5. Implement sliding-window, summarization, and (if available) server-side compaction for a 100-turn synthetic chat. Test **fact survival** with 20 probe questions each.
- [ ] 🟡 E6. Tool-result bloat: an agent that calls a verbose search tool; measure context growth; fix via truncation/pagination/context editing; compare cost and accuracy.
- [ ] 🔴 E7. Build `llm-budget`: given a feature's token budget table (system/tools/history/retrieval/output) enforce limits in code, with graceful degradation (truncate → summarize → error).
- [ ] 🔴 E8. Cost model for a real scenario (e.g., 50k support tickets/day): compute daily/monthly cost with/without caching, batch, routing; validate against a 1% pilot using real usage logs.
- [ ] 🔴 E9. Long-context study: place a fact at 10 positions in 20k/60k/150k-token contexts; plot recall vs position/length; compare with RAG over the same docs.
- [ ] 🔴 E10. Build a **memory** system for a personal assistant: notes file/DB the model reads/writes via tools; add dedupe, expiry, and conflict resolution; eval: does it recall a preference after 50 unrelated turns?

## F. Multimodal & documents (Note 06)
- [ ] 🟢 F1. Send 10 receipt photos; extract merchant/date/total as JSON; hand-verify; compute field accuracy.
- [ ] 🟡 F2. PDF Q&A with citations: ask 20 questions of a contract; verify each cited span supports the answer; compute citation precision.
- [ ] 🟡 F3. Image token cost experiment: same photo at 4 resolutions; plot tokens, latency, accuracy.
- [ ] 🟡 F4. Native-PDF vs page-images vs OCR-text-first for a table-heavy report; compare accuracy/cost.
- [ ] 🟡 F5. Upload a CSV and use code execution to compute stats and make a chart; download the generated files safely.
- [ ] 🔴 F6. Build the **doc-intake pipeline**: classify → type-specific schema → extract → validate (totals/dates/IDs) → confidence → review queue UI → write to DB. Evaluate straight-through rate and field F1 on 100 docs.
- [ ] 🔴 F7. Build a voice loop: STT (Whisper/API) → LLM (streaming) → TTS; measure end-to-end latency per stage; optimize.
- [ ] 🔴 F8. Redaction layer: detect & mask PII (Presidio or regex+LLM) before sending docs to an API; verify with a labeled set; discuss residual risk.

## G. Providers & patterns (Note 07)
- [ ] 🟡 G1. Port one feature (extraction w/ schema) to a second provider; list every code change required; run both on your eval set.
- [ ] 🟡 G2. Put LiteLLM (or another gateway) in front of two providers; add budget limits and a fallback rule; inspect logs.
- [ ] 🟡 G3. Call Claude (or another model) via a cloud platform client (Bedrock/Vertex/Foundry) in a free-tier/sandbox account; document auth, model-ID, and feature differences.
- [ ] 🔴 G4. Build a feature-flagged **model migration**: run old and new model in shadow on 1,000 real-like requests, diff outputs with an LLM judge + metrics, and write a go/no-go memo.
- [ ] 🔴 G5. Threat-model a customer-support copilot with tools (refunds, account lookup): enumerate prompt-injection paths, define least-privilege tool scopes, approval gates, logging; write a 2-page security note.

## H. Mini-capstones (pick 2)
- [ ] 🔴 H1. **Meeting notes → action items** service: audio in, structured tasks out, Slack/email integration, eval set of 30 meetings.
- [ ] 🔴 H2. **Email triage assistant**: classify, extract entities, draft replies with citations to policy docs, human approval queue.
- [ ] 🔴 H3. **Invoice processor** (see Project 3 in [[11-Projects-and-Checkpoint]]).

## Cadence
Daily 1–2 exercises; weekly write-up of a surprising finding; Sunday Q&A review ([[09-QA-Interview-Questions]]).
