---
tags: [phase-4, streaming, async, batch, reliability]
---
# 04 — Streaming, Async, Batch & Reliability

> LLM calls are **slow (seconds to minutes), expensive, rate-limited, and non-deterministic**. This note is the engineering that makes them production-safe.

## 1. Streaming
**Why:** perceived latency (time-to-first-token), long outputs without HTTP timeouts, progressive UIs, early cancellation. Transport is **Server-Sent Events (SSE)** ([[../Phase-1-Foundations/06-HTTP-APIs-JSON]]).

```python
with client.messages.stream(
    model="claude-opus-5-5", max_tokens=64000,
    messages=[{"role": "user", "content": "Write a story"}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
    final = stream.get_final_message()          # accumulated Message: usage, stop_reason, all blocks
```
- **Rule of thumb:** *default to streaming for long inputs/outputs or large `max_tokens`*; the SDK refuses huge non-streaming requests it expects to exceed ~10 min. `get_final_message()` gives you the complete response if you don't need events.
- **Event sequence:** `message_start` → (`content_block_start` → `content_block_delta`* → `content_block_stop`)* → `message_delta` (stop_reason, usage) → `message_stop`. Delta types: `text_delta`, `thinking_delta`, `input_json_delta` (tool-call args arrive as partial JSON).
- **Streaming + tools:** accumulate partial tool JSON; **don't execute until the block completes and validates**; with eager tool-input streaming the server no longer guarantees valid/complete JSON, so validate against your schema and check `max_tokens` before running (truncated input can look valid). Confine any model-provided file path to a root.
- **Streaming thinking:** thinking text is omitted by default; request `display: "summarized"` to show reasoning summaries (a long silent pause otherwise).
- **Interruption handling:** network drops mid-stream → you may hold partial content; design retry vs resume; for user-initiated cancel, close the stream (stop paying for tokens).
- **Backpressure/UI:** buffer tokens, render on animation frames, handle markdown mid-token, auto-scroll.

### Serving a stream to a browser (FastAPI SSE)
```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import anthropic, json
app = FastAPI(); aclient = anthropic.AsyncAnthropic()

@app.post("/chat")
async def chat(payload: dict):
    async def gen():
        async with aclient.messages.stream(model="claude-sonnet-5-5", max_tokens=4000, messages=payload["messages"]) as s:
            async for t in s.text_stream:
                yield f"data: {json.dumps({'delta': t})}\n\n"
            yield "data: [DONE]\n\n"
    return StreamingResponse(gen(), media_type="text/event-stream")
```
Client: `EventSource` / `fetch` + `ReadableStream`. Mind proxies that buffer (disable buffering on nginx), set keep-alive heartbeats, handle reconnect with `Last-Event-ID`, and **never expose your API key client-side**.

## 2. Async & concurrency (Phase 1 asyncio pays off)
Most LLM workloads are **I/O-bound** → `asyncio` with a **semaphore** to respect rate limits.
```python
import asyncio, anthropic
aclient = anthropic.AsyncAnthropic(max_retries=4, timeout=120)
sem = asyncio.Semaphore(8)

async def classify(text: str) -> str:
    async with sem:
        r = await aclient.messages.create(
            model="claude-haiku-5-5", max_tokens=50, output_config={"effort": "low"},
            system="Classify the ticket as billing|bug|feature|other. Reply with the label only.",
            messages=[{"role": "user", "content": text}])
        return next(b.text for b in r.content if b.type == "text").strip()

async def main(texts):
    return await asyncio.gather(*(classify(t) for t in texts), return_exceptions=True)
results = asyncio.run(main(texts))
```
- **`return_exceptions=True`** (or `TaskGroup` with per-task try/except) so one failure doesn't cancel the batch; collect failures for retry/dead-letter.
- Choose concurrency from **rate limits (RPM/TPM)**, not CPU. Adaptive concurrency: reduce on 429, probe upward slowly (AIMD).
- Token-aware rate limiting: track estimated tokens/min (use `count_tokens` or `len(text)/4` heuristic) and sleep to stay under TPM.
- For very high concurrency, use the SDK's aiohttp backend (`anthropic[aiohttp]`).
- **Checkpoint progress** (write results as they complete, resume by ID) — a crashed 10k-item job shouldn't restart from zero.
- **Idempotency:** LLM calls themselves are safe to retry (no side effects) but **tool calls with side effects are not** — use idempotency keys.

## 3. Batch API (offline, ~50% cheaper)
For non-latency-sensitive bulk work (evals, backfills, labeling, nightly enrichment), providers offer **batch endpoints**: submit up to thousands of requests, results within hours (typically ≤24h), at a **discount (Anthropic: 50%)**. Gemini and OpenAI have analogous batch APIs (Gemini added webhooks to replace polling in 2026).
```python
batch = client.messages.batches.create(requests=[
    {"custom_id": f"doc-{i}", "params": {"model": "claude-haiku-5-5", "max_tokens": 1024,
        "messages": [{"role": "user", "content": f"Summarize: {d}"}]}}
    for i, d in enumerate(docs)])
# poll until processing_status == "ended", then:
for entry in client.messages.batches.results(batch.id):          # ORDER IS NOT GUARANTEED → key by custom_id
    if entry.result.type == "succeeded":
        out[entry.custom_id] = entry.result.message.content
    else: failures[entry.custom_id] = entry.result.type          # errored / canceled / expired
```
- Combine with **prompt caching** for shared prefixes (big savings on evals).
- Not for interactive UIs; check which features/models batch supports (some parameters, fast mode and priority tier don't apply).
- Failure handling: partial success is normal; re-submit failed `custom_id`s.

## 4. Retries, timeouts, and backoff (done right)
- **Retry:** connection errors, 408, 409 (sometimes), **429**, **5xx/529**. **Don't retry** 400/401/403/404 (fix the request).
- **Exponential backoff + jitter**; honor `retry-after`. The SDK already does this (default 2 retries) — raise `max_retries` for batch jobs, lower for interactive paths.
- **Timeouts:** connect/read; interactive endpoints 30–60 s with streaming; background jobs longer. Total wall-clock can reach `timeout × (retries+1)` — budget for it.
- **Deadline propagation:** pass an overall deadline down your call stack; stop retrying when the user has gone.
- **Circuit breaker:** after N consecutive failures stop calling for a cool-down window; fail fast to a fallback.
- **Bulkheads:** separate concurrency pools per feature/provider so a runaway job can't starve interactive traffic.
- **Hedged requests** (send a second request if the first is slow) — reduces tail latency, doubles cost on slow calls; use sparingly.

## 5. Fallbacks, routing & multi-model strategies
- **Provider outage/overload fallback:** primary model → secondary model (same provider tier down, or another provider). Normalize prompts/outputs behind an interface ([[07-Multi-Provider-and-App-Patterns]]); re-validate outputs because behavior differs.
- **Refusal fallback:** some providers offer server-side fallback on safety-classifier refusals (Anthropic: opt-in `fallbacks` param with a beta header on top-tier models; client-side middleware on cloud partners). Read the semantics (billing, streaming).
- **Routing:** cheap classifier/small model decides easy vs hard → send hard queries to the flagship. Measure **cost per completed task** and quality; mind that caches are **per model**, so cascades forfeit cache reuse.
- **Cascades with verification:** small model answers; verifier/confidence check; escalate on low confidence.
- **Speed tiers:** **fast mode** (same model, higher output tokens/sec at premium price — limited models/platforms), **priority tier** (committed capacity; not available for all current models). **Measure before paying.**
- **Degraded modes:** cached/stale answer, shorter answer, "try again", human handoff.

## 6. Rate limits & capacity planning
- Know your tier's **RPM/ITPM/OTPM**; read `anthropic-ratelimit-*`-style response headers (names vary by provider).
- Capacity estimate: `peak_rps × avg_tokens_per_request` vs TPM; add headroom; request limit increases early (days of lead time).
- **Queue + workers** for background jobs (Celery/RQ/SQS/Temporal); autoscale on queue depth; **dead-letter queues**; **durable execution** (Temporal, Inngest, Restate) for long multi-step LLM workflows.
- **Cost guards:** per-user/per-tenant token budgets, daily caps, alerts on spend anomalies, max steps for agent loops.

## 7. Determinism, caching of results, and idempotency
- **Don't expect determinism.** Even with low randomness there is variation; design with tolerance, and evaluate distributions ([[../Phase-2-ML-Deep-Learning/04-Evaluation-and-Metrics]]).
- **Application-level response cache** (key = hash(model, prompt version, normalized inputs)) for repeated identical requests; **semantic cache** (embedding similarity) is riskier — only for tolerant use cases, with TTLs and invalidation.
- Provider **prompt caching** is separate (prefix reuse inside the provider) → Note 05.
- Idempotent job IDs; exactly-once is a myth — design for at-least-once + dedup.

## 8. Observability starter kit (full treatment in Phase 7)
Per call log: request ID, model, prompt version, input/output/cached tokens, latency (TTFT & total), stop_reason, retries, cost, user/tenant, feature, success/failure class. Use **OpenTelemetry** spans (SDKs ship instrumentation hooks), LLM-observability tools (Langfuse, LangSmith, Braintrust, Phoenix, Helicone), and dashboards for p50/p95/p99 latency, error rates, cost per feature, cache hit rate.

## 9. Resilient client pattern (put it together)
```python
import asyncio, random, anthropic

PRIMARY, FALLBACK = "claude-sonnet-5-5", "claude-haiku-5-5"
aclient = anthropic.AsyncAnthropic(max_retries=2, timeout=60)

async def resilient(messages, system=None, deadline_s=45):
    for model in (PRIMARY, FALLBACK):
        try:
            return await asyncio.wait_for(
                aclient.messages.create(model=model, max_tokens=2000, system=system, messages=messages),
                timeout=deadline_s)
        except (anthropic.RateLimitError, anthropic.APIConnectionError, asyncio.TimeoutError):
            await asyncio.sleep(random.uniform(0.2, 1.0)); continue
        except anthropic.APIStatusError as e:
            if e.status_code < 500: raise
            continue
    raise RuntimeError("all models failed")
```

## Self-check
1. When is streaming mandatory? 2. Why `return_exceptions=True`? 3. Why can't you key batch results by order? 4. Which errors should/shouldn't be retried? 5. Why do model cascades hurt prompt-cache hit rates? 6. What do you log for every call?

Next → [[05-Context-Engineering-Caching-Cost]]
