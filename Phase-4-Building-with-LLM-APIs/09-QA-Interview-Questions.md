---
tags: [phase-4, qa, interview]
---
# 09 — Q&A (Phase 4)

> Answer aloud first. Where a question touches a fast-moving API detail, the answer states the principle and flags what to verify.

## API fundamentals
> [!question]- Q1. Why are LLM APIs stateless and what does that imply?
> Each request is independent; you must resend the conversation (system prompt, history, tools). Consequences: cost and latency grow with history, you own memory/persistence, and you can use caching/compaction/retrieval to manage it.

> [!question]- Q2. What should you log on every LLM call?
> Request ID, model, prompt/template version, input/output/cached/thinking tokens, latency (TTFT & total), stop_reason, retries, cost, tenant/feature, success/failure class (and raw response for debugging, subject to PII policy).

> [!question]- Q3. What are the common `stop_reason` values and why do they matter?
> `end_turn` (complete), `max_tokens` (truncated — invalid JSON/partial answers), `stop_sequence`, `tool_use` (execute tool), `pause_turn` (resume server-tool turn), `refusal` (safety decline). Ignoring them causes silent truncation and broken parsers.

> [!question]- Q4. Why not assume `response.content[0].text`?
> Content is a list of typed blocks — thinking, tool_use, server-tool results, citations — so the first block may not be text. Iterate and branch on block type.

> [!question]- Q5. Which errors do you retry?
> Connection errors, timeouts, 408, 429 (honor `retry-after`), and 5xx/overloaded — with exponential backoff and jitter. Not 400/401/403/404 (fix the request).

> [!question]- Q6. Why can't you rely on `temperature=0` in the Claude API today?
> Current Claude models reject sampling parameters (non-default values return 400; Anthropic's docs say requests sending them 400 on every model from 2026-10-21), so behavior is steered via prompts, effort, and structured outputs. Other providers/open models still expose them. Never assume determinism anyway.

> [!question]- Q7. What replaced `budget_tokens` for extended thinking?
> Adaptive thinking (`thinking: {type: "adaptive"}`) plus `output_config.effort` (`low`…`max`). The model decides how much to think; effort sets the trade-off. Fixed budgets are rejected on recent models. Reasoning tokens are billed as output.

> [!question]- Q8. Why is token counting provider-specific?
> Different tokenizers produce different counts for the same text. Use the provider's count endpoint (e.g., `count_tokens`), not another vendor's tokenizer, for budgeting and cost.

> [!question]- Q9. Where do API keys belong?
> Server-side only, in environment variables or a secret manager; never in client apps or repos; separate dev/prod keys; rotate; set spend limits; prefer cloud IAM/workload identity in enterprise.

> [!question]- Q10. Output tokens vs input tokens — cost implications?
> Output is priced several times higher and generated sequentially; reasoning tokens count as output. So cap output length, use concise formats/structured outputs, and tune effort; caching mainly reduces input cost.

> [!question]- Q11. How do you pick a model tier?
> Establish the quality ceiling with the strongest model on your eval set, then step down (mid/small, lower effort, routing) while holding quality; consider latency, cost, context, tool reliability, privacy/compliance, and rate limits.

## Prompting
> [!question]- Q12. What makes a good prompt?
> Clear task, audience/purpose, relevant context, success criteria, output format, examples for edge cases, permission to abstain, and structured sections — validated against a test set rather than intuition.

> [!question]- Q13. Few-shot vs instructions — when?
> Examples help most for format/style/labeling nuance and ambiguous boundaries. Use diverse, representative examples; consistent format. Often 3–5 suffice; test order/balance sensitivity.

> [!question]- Q14. Why put long documents before the question?
> Models tend to use queries at the end more effectively over long context; placing documents first and instructions/question last improves quality and keeps stable content at the front for caching.

> [!question]- Q15. Why might older, heavily scaffolded prompts hurt newer models?
> Newer models follow instructions more literally and reason better on their own; excessive step-by-step directives, shouted rules, and redundant reminders can make behavior rigid or over-triggered. Prune and re-evaluate after each model upgrade.

> [!question]- Q16. How do you control output format without assistant prefill?
> Schema-constrained structured outputs, explicit format sections with examples, XML-tag wrapping, stop sequences, and validation/retry.

> [!question]- Q17. Describe a rigorous prompt-iteration workflow.
> Define success + build a test set (edge/adversarial/abstain cases) → baseline → error analysis → change one variable → rerun everything → track prompt version × metrics × cost/latency → hold out validation → ship with monitoring and feed real failures back into tests.

> [!question]- Q18. When should you chain prompts instead of using one big prompt?
> When tasks have distinct steps, need intermediate inspection/validation, mix tools/code between steps, or the single prompt's quality is inconsistent. Chaining improves debuggability and per-step evaluation at the cost of latency/calls.

> [!question]- Q19. How do you reduce hallucination at the prompt/system level?
> Provide grounding sources, require quotes/citations, allow "not found," constrain outputs, use tools for facts/calculation, verify outputs, and evaluate on abstention cases. Prompting alone is insufficient.

## Structured outputs & tools
> [!question]- Q20. What does schema-constrained output guarantee?
> Syntactically valid output conforming to the schema (decoding is masked). It doesn't guarantee factual correctness or semantic validity — validate business rules separately. Check `stop_reason` for truncation/refusal.

> [!question]- Q21. How do you design a good schema?
> Descriptive field docs, enums for closed sets, nullable fields with "don't guess," an "other/unknown" option, evidence/rationale fields placed before the decision field, shallow structure, and semantic validation in code.

> [!question]- Q22. Walk through the tool-use loop.
> Send messages + tool definitions → model returns `tool_use` blocks (stop_reason `tool_use`) → your code validates/authorizes and executes → append the assistant content, then one user message containing all `tool_result` blocks with matching IDs → call again → repeat until `end_turn`, with loop limits.

> [!question]- Q23. Why return all tool results in one message?
> The API expects results for the tool calls from one assistant turn together; splitting across messages can discourage the model from making parallel calls and may break conventions.

> [!question]- Q24. What should a tool error look like?
> `is_error: true` with an actionable, concise message suggesting how to recover (e.g., valid formats or next step), not a stack trace.

> [!question]- Q25. What makes tool descriptions effective?
> What it does, when to use it and when not, parameter semantics and formats, return value, side effects, limits — written as an instruction to a smart new colleague. Few high-level tools beat many thin wrappers.

> [!question]- Q26. How do you get guaranteed JSON now that forced tool choice is removed on the newest Claude models?
> Use structured outputs (`output_config.format`/`parse`) for the final answer; use `auto` tool choice + `strict: true` tools + explicit instructions for tool calls, and verify a tool block was returned.

> [!question]- Q27. Why must you validate model-provided tool arguments?
> The model can be wrong or manipulated (prompt injection). Authorize IDs against the user's permissions, confine file paths to a root, parameterize queries, enforce limits, and require confirmation for destructive actions.

> [!question]- Q28. What is MCP?
> The Model Context Protocol: an open standard for exposing tools, resources and prompts from servers to LLM apps, so integrations are written once and reused across clients; can be used via SDK helpers or provider connectors.

> [!question]- Q29. Client tools vs server tools?
> Client tools run in your code (you execute & return results). Server tools (web search/fetch, code execution) run on the provider's infrastructure; results arrive in the same response. Server-tool errors may appear in result blocks with HTTP 200.

## Streaming, reliability
> [!question]- Q30. When should you stream?
> Interactive UIs (low perceived latency), long outputs/large `max_tokens` (avoid timeouts), and when you want early cancellation. Use `get_final_message()` to obtain the full response if events aren't needed.

> [!question]- Q31. What are the dangers of streaming tool-call arguments?
> Partial/invalid JSON or truncated inputs that look valid; execute only after block completion + schema validation + `stop_reason` check; never run on truncated input.

> [!question]- Q32. How do you process 10,000 documents efficiently?
> If offline: Batch API (~50% cheaper, async) with `custom_id`s and failure resubmission; if near-real-time: async with a semaphore tuned to rate limits, retries/backoff, checkpointing. Add caching for shared prefixes and routing to cheaper models.

> [!question]- Q33. Why `asyncio.gather(..., return_exceptions=True)`?
> So one failed call doesn't cancel the rest; you collect exceptions to retry or dead-letter.

> [!question]- Q34. Design a robust LLM client.
> Timeouts, retries with backoff+jitter honoring `retry-after`, circuit breaker, concurrency limits, request IDs/logging, fallback model/provider, validation of outputs, budget guards, and idempotency for side-effecting tools.

> [!question]- Q35. Model cascade pitfalls?
> Caches are per-model so cascades lose reuse; added latency; routing errors; complexity. Test "strongest model at lower effort" first and measure cost per completed task.

> [!question]- Q36. How do you handle rate limits at scale?
> Know RPM/TPM tiers, token-aware limiter, queues + worker autoscaling, backoff, multi-key/region/provider strategies where allowed, request limit increases early, and per-tenant quotas.

## Context & cost
> [!question]- Q37. Define context engineering.
> Curating exactly which tokens the model sees each call — instructions, tools, history, retrieved data, memory — to maximize quality within cost/latency limits, via JIT retrieval, compaction, isolation (sub-agents), and persistence outside context.

> [!question]- Q38. What is context rot?
> Degradation in recall/reasoning as context grows and fills with distractors; mitigated by shorter, cleaner context, retrieval, compaction, and ordering.

> [!question]- Q39. How does prompt caching work and how do you verify it?
> The provider reuses processed KV state for an identical prompt prefix, charging reads at a fraction of input price and cutting TTFT. Put stable content first, volatile last, set breakpoints; verify via `usage` cache-read tokens being non-zero across repeated requests.

> [!question]- Q40. Name five silent cache invalidators.
> Timestamps/UUIDs in the system prompt, unsorted/non-deterministic JSON serialization, changing tool lists/order, per-user content before the stable prefix, switching model or relevant request settings, and TTL expiry between calls.

> [!question]- Q41. Compaction vs context editing vs memory?
> Compaction summarizes older context (lossy); context editing deletes specified stale blocks like old tool results (cheap, deterministic); memory stores durable notes outside the window for later retrieval. Use combinations and test fact retention.

> [!question]- Q42. A conversation with compaction forgets things after many turns. Likely bug?
> Appending only the response text instead of the full `response.content`, dropping the compaction block the API needs to replace old history.

> [!question]- Q43. How do you estimate the cost of a feature?
> Build per-request token profile (uncached/cache-write/cache-read/output+thinking) × prices × requests per task (incl. retries/tool loops/judge calls) × volume; validate on a pilot using real `usage` logs; include overhead for failures.

> [!question]- Q44. Order the cost levers.
> Prompt caching → trim inputs/tool results → loop hygiene → shorter outputs → Batch API (offline) → lower effort per route → smaller model/routing (eval-validated) → fine-tuning/distillation for high volume.

> [!question]- Q45. Why do tool results often dominate context cost?
> Verbose payloads accumulate every loop iteration and persist in history; fix by concise/paginated results, summarizing, clearing stale results, or processing data in a code sandbox.

## Multimodal & providers
> [!question]- Q46. How do image inputs affect cost/latency?
> Image tokens scale with resolution/tiling; downscale/crop to needed detail; count tokens; use cache for repeated images/documents.

> [!question]- Q47. How do you make document extraction reliable?
> Classify doc type, type-specific schemas, native-PDF/vision/OCR evaluation, evidence quotes, business-rule validation (totals/dates/IDs), confidence/disagreement checks, human review queue, golden set with field-level metrics and slices.

> [!question]- Q48. Why can't citations be combined with schema-constrained output on Claude, and what's the workaround?
> The API rejects the combination (400) because citation interleaving conflicts with constrained JSON. Use two passes (answer with citations, then structure) or include quote/location fields in the schema.

> [!question]- Q49. When RAG instead of putting the whole PDF in context?
> When the corpus is large/changing, per-question relevance is sparse, cost/latency matter, permissions vary per user, or you need ranking control/citations at scale; long-context is fine for a single bounded document often asked about repeatedly (with caching).

> [!question]- Q50. What breaks when switching providers?
> Tool-call formats/IDs, system prompt placement, token/limit semantics, thinking/reasoning controls, schema subset, stop/refusal signaling, streaming events, rate-limit headers, safety behavior — hence keep evals and a thin adapter layer.

> [!question]- Q51. Gateway vs thin internal wrapper?
> Gateways (LiteLLM/OpenRouter/etc.) give unified API, routing, budgets and logs but flatten features and add a dependency/hop; a thin internal interface keeps control and lets you use provider-specific features. Many teams use both.

> [!question]- Q52. Why use Claude via Bedrock/Vertex/Foundry?
> IAM/VPC/private networking, data residency, consolidated billing, compliance posture; trade-offs: delayed feature availability, different quotas/model IDs/clients.

## Architecture & safety
> [!question]- Q53. Workflow vs agent — how do you choose?
> Start with the simplest working tier: single call → code-defined workflow → autonomous agent only when steps can't be predetermined and error cost is tolerable; agents add cost, latency, and unpredictability.

> [!question]- Q54. Design a safe text-to-SQL feature.
> Read-only DB role, allow-listed views, schema + glossary in context, SQL parsing/validation, row/time limits, EXPLAIN dry run, show SQL, sanity checks, audit logs, evals on gold questions, human approval for sensitive data.

> [!question]- Q55. What is prompt injection and first-line defenses?
> Untrusted content (web, docs, tool results) containing instructions that hijack behavior. Defenses: delimit/label untrusted data, least-privilege tools, human approval for risky actions, output validation, isolating sub-agents, monitoring; no defense is perfect — design for blast-radius control.

> [!question]- Q56. How would you roll out a new model version safely?
> Re-run the eval suite, shadow traffic comparison with LLM-judge/metrics, canary a small percentage with monitoring (quality, cost, latency, refusals), prune old prompt cruft, have rollback via config flags, and check deprecation timelines.

> [!question]- Q57. A customer says "the AI is wrong sometimes — can we trust it?" Respond.
> Quantify error with an eval on their data (not anecdotes), show per-slice accuracy and failure types, design human review for low-confidence/high-impact cases, add citations and validations, define monitoring and an improvement loop; set expectations on residual risk.

> [!question]- Q58. How would you cut a $50k/month LLM bill by 50% without hurting quality?
> Profile tokens per feature → enable/verify caching → trim prompts/tool results → shorten outputs → move offline work to Batch → lower effort/model on easy routes validated by evals → dedupe/cache repeated requests → consider fine-tuning for the top high-volume task; measure per-task cost and quality after each step.

> [!question]- Q59. Design a chat feature's memory.
> Short-term: windowed/compacted history with cache-friendly ordering; long-term: user facts/notes in a DB or memory tool with dedupe/expiry/conflict rules; retrieve selectively per turn; allow user view/delete; evaluate recall after many turns.

> [!question]- Q60. What's the first thing you do when an LLM feature "got worse" in production?
> Check what changed (model version, prompt, data, provider incidents), review logs/traces for the failing cases, reproduce with saved inputs, run the eval suite against current vs previous versions, look at distribution shift and tool/retrieval failures, then roll back via config while fixing the root cause.

Next → [[10-Resources-and-Practice-Links]]
