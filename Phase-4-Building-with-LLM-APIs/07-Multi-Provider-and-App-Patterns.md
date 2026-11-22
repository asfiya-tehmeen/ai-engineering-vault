---
tags: [phase-4, providers, patterns, architecture]
---
# 07 — Multi-Provider Strategy & Common App Patterns

## 1. Why think multi-provider?
Outages, rate limits, price changes, model deprecations, customer mandates ("must run on Azure/Bedrock/Vertex"), data-residency, and **capability differences**. As an FDE you'll meet customers locked into one cloud — your code must adapt without a rewrite.

## 2. Same ideas, different shapes
| Concept | Anthropic | OpenAI | Google Gemini |
|---|---|---|---|
| Primary API | Messages | Responses (newer) / Chat Completions | `generateContent` / newer Interactions API (schema migrating — check docs) |
| System instructions | top-level `system` | `instructions` / system-developer role | `system_instruction` |
| Content | typed blocks (`text`, `image`, `document`, `tool_use`…) | typed items (messages, tool calls & outputs as separate items in Responses) | `parts` |
| Tools | `tools` + `tool_use`/`tool_result` | `tools` + function call items; `strict: true` | function declarations; built-in tools combinable (2026) |
| Structured output | `output_config.format` / `parse` | Structured Outputs (`strict` JSON Schema), `parse` helper | `response_schema` / structured output with tools (Gemini 3) |
| Reasoning control | adaptive thinking + `effort` | reasoning effort; reasoning items reusable across calls | `thinking_level` (Gemini 3) |
| Caching | `cache_control` breakpoints / auto | automatic prefix caching | implicit + explicit context caching |
| Batch | Message Batches (−50%) | Batch API | Batch API (+ webhooks) |
| Files | Files API | Files API / vector stores | Files API |
| Refusals | `stop_reason: "refusal"` + `stop_details` | separate `refusal` field | safety ratings / block reasons |
(Treat this table as orientation — verify each cell in current docs.)

## 3. Abstraction strategies
1. **Thin internal interface** (recommended): your own `LLMClient.complete(messages, schema=None, tools=None, ...) -> Result` with provider adapters. You control semantics (retries, logging, cost), and can use provider-specific features behind flags.
2. **OpenAI-compatible gateway/proxy**: **LiteLLM**, **OpenRouter**, Portkey, Cloudflare AI Gateway, Kong/Envoy AI gateways, or cloud-native (Bedrock/Vertex). Pros: one API, routing, budgets, logging, caching, fallbacks, key management. Cons: lowest-common-denominator features, translation quirks (tool/thinking/caching fields), extra hop, trust in a third party.
3. **Frameworks**: LangChain/LangGraph, LlamaIndex, Pydantic AI, Vercel AI SDK, Instructor, DSPy, Haystack. Pros: speed, ecosystem. Cons: leaky abstractions, churn, hidden prompts — *learn raw APIs first (this phase), adopt frameworks deliberately in Phase 6.*
4. **Don't abstract too early.** The prompts and evals are what's portable; test every provider/model on your eval set — behavior differs.

Gotchas when switching: tool-call format/IDs, parallel tool-call semantics, system-prompt placement, max-token semantics (does it include thinking?), stop reasons, token counting, safety/refusal behavior, JSON-schema subset, image limits, streaming event shapes, rate-limit headers, error classes.

## 4. Deploying through cloud platforms (FDE-relevant)
- **AWS Bedrock**, **Google Vertex AI**, **Microsoft Foundry/Azure OpenAI**: models behind your cloud account → IAM auth, private networking (VPC endpoints/Private Link), regional data residency, consolidated billing, compliance attestations. Trade-offs: new features arrive later, different quotas/model IDs/clients (dedicated client classes — not the first-party client with a `base_url` hack), differing availability of tools/betas.
- **Self-hosted open-weight** (vLLM/TGI/Ollama): full control and air-gap capability; you own ops, scaling, safety, and GPU cost ([[../Phase-3-Transformers-LLM-Internals/05-Inference-and-Decoding]]).
- **Decision grid:** data sensitivity & residency → capability needs → latency → volume/cost → team ops capacity.

## 5. Reference application patterns
### 5.1 Chat assistant (with memory)
System prompt + history management + streaming UI + usage tracking + safety. Add: conversation store (DB), title generation (small model), feedback buttons (👍/👎 → eval data), per-user rate limits, moderation, export/delete (privacy).
### 5.2 Extraction/ETL ("unstructured → structured")
Prompt + schema + validation + retries + human review + idempotent writes. Batch API for backfills. Eval with field-level metrics. (Most common enterprise use.)
### 5.3 Classification & routing
Small model, low effort, enum output, few-shot, confidence/"other". Use for ticket triage, intent routing, content moderation, model routing. Calibrate thresholds; log disagreements for review.
### 5.4 Summarization/report generation
Define audience, purpose, length, must-include; map-reduce for long inputs; cite sources; evaluate faithfulness (no unsupported claims) not just fluency.
### 5.5 Text-to-SQL / analytics copilot
Give schema + sample rows + business glossary; **read-only DB user**, row limits, query timeout, allow-list of tables/views, SQL parsing/validation (sqlglot), dry-run `EXPLAIN`, show SQL to user, result sanity checks. Evaluate execution accuracy on a gold question set. Never connect with write privileges.
### 5.6 Code assistant / generation
Tools for reading files/running tests, sandboxed execution, small diffs, test-driven loop; human review/CI gates. (Agentic coding → Phase 6.)
### 5.7 Semantic search & Q&A over documents
→ RAG (Phase 5).
### 5.8 Workflow automation / agents
LLM chooses tools across steps with guardrails → Phase 6.
### 5.9 Content moderation & safety classification
Dedicated prompts/classifiers; layered with provider safety; human escalation.
### 5.10 Translation/localization, tone rewriting, data enrichment, entity resolution, PII redaction, meeting notes, email drafting, etc. — all variants of the patterns above.

## 6. Workflow vs agent (simplicity rule)
Prefer the **simplest** architecture that works:
1. **Single call** (+ retrieval/few-shot) →
2. **Workflow**: code-defined chain/branches/parallel steps (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer) →
3. **Agent**: model-driven loop choosing tools autonomously (use only when steps can't be predetermined and error cost is manageable).
Higher tiers = more cost, latency, and unpredictability. (Phase 6 details.)

## 7. Safety & compliance basics for app builders
- **Input/output moderation** layers; allow-lists for tools/domains; PII detection/redaction (Presidio, cloud DLP) before logging or sending where policy demands.
- **Prompt-injection hygiene**: delimit untrusted content, least-privilege tools, human approval for sensitive actions, output validation (Phase 7).
- **Data governance**: what leaves your boundary? Provider retention/training policies, **zero-data-retention** options (some features/models are ineligible), DPAs, regional endpoints, customer-managed keys.
- **Transparency**: tell users they're talking to AI where required; show sources; provide feedback/appeal paths; keep human oversight for consequential decisions (EU AI Act / sector rules).
- **Abuse prevention**: auth, per-user quotas, input length caps, bot protection.

## 8. UX patterns for LLM features
Streaming with stop button · skeleton/"thinking" states · **show sources/citations** · editable outputs · undo for actions · confirmations for irreversible ops · feedback capture · graceful errors with retry · progressive disclosure of reasoning · explicit uncertainty ("I couldn't find this in your documents") · latency masking via optimistic UI · cost controls visible to admins.

## 9. Project hygiene
- Config-driven model IDs and parameters (env/feature flags) — models get deprecated; plan **migration playbooks**.
- Prompts in files with versions; unit tests for parsing/validation; **eval suite in CI** (Phase 7); golden datasets; canary new models; shadow traffic.
- Pin SDK versions; read changelogs; subscribe to provider deprecation notices.
- Cost/latency dashboards per feature; budgets & alerts.

## Self-check
1. When would you choose a gateway vs a thin internal interface? 2. List 6 things that break when swapping providers. 3. Design guardrails for a text-to-SQL feature. 4. Why start with the simplest architecture tier? 5. What are the trade-offs of using Claude through Bedrock/Vertex vs first-party?

Next → [[08-Exercises]]
