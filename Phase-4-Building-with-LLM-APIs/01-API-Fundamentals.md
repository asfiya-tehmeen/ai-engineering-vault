---
tags: [phase-4, api, fundamentals]
---
# 01 — LLM API Fundamentals

## 1. The mental model
An LLM API is **one stateless HTTP endpoint**: you send a list of messages (+ options), the server runs prefill + decode ([[../Phase-3-Transformers-LLM-Internals/05-Inference-and-Decoding]]) and returns generated content plus **usage** (token counts).
- **Stateless:** the server doesn't remember earlier calls. *You* resend the conversation each time (some providers offer server-side state/threads as opt-in conveniences — see the end of this note). Therefore **cost grows with conversation length** (every turn re-sends history; prompt caching mitigates this).
- Everything in the request is **context**: system prompt, tools, history, documents, images. All of it is tokens you pay for and the model attends to.

## 2. Anatomy of a Messages request (Anthropic shape)
```python
import anthropic
client = anthropic.Anthropic()      # reads ANTHROPIC_API_KEY (or other credentials) from the environment

response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,                                   # REQUIRED upper bound on generated tokens
    system="You are a concise support assistant.",      # operator instructions (top-level, not a message)
    messages=[                                          # alternating user/assistant turns; first must be user
        {"role": "user", "content": "How do I reset my password?"},
    ],
)
for block in response.content:                          # content is a LIST of typed blocks
    if block.type == "text":
        print(block.text)
print(response.stop_reason, response.usage)
```
Key points:
- **`model`** — exact ID, no date suffix invented by you. IDs in the Claude 5 family at time of writing: `claude-opus-5-5`, `claude-sonnet-5-5`, `claude-haiku-5-5`, `claude-fable-5-2` (check the Models API / docs for current).
- **`max_tokens`** — hard cap on output (including thinking). Too low → truncated output (`stop_reason == "max_tokens"`). Too high costs nothing *unless used*, but for non-streaming requests very large values can trip SDK timeouts → **stream** large outputs.
- **`system`** — persistent instructions/persona/rules. Put stable material here (good for caching).
- **`messages`** — list of `{role, content}`. `content` is a string or a list of blocks (text, image, document, tool_use, tool_result, thinking…). The API may merge consecutive same-role messages.
- **Response `content`** — list of blocks: `text`, `thinking`, `tool_use`, server-tool blocks… **Never assume `content[0]` is text.**
- **`usage`** — `input_tokens`, `output_tokens`, plus cache read/write counts. **Log this on every call** — it's your cost and debugging data.
- **`response.id` / `_request_id`** — log for support/debugging.
- **`stop_reason`:** `end_turn` (done), `max_tokens` (truncated!), `stop_sequence`, `tool_use` (run the tool), `pause_turn` (server tool paused; continue), `refusal` (safety decline; check `stop_details`). **Always branch on `stop_reason`** — silent truncation is a top production bug.

### Raw HTTP (know what the SDK does)
```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" -H "anthropic-version: 2023-06-01" -H "content-type: application/json" \
  -d '{"model":"claude-haiku-5-5","max_tokens":256,"messages":[{"role":"user","content":"Hi"}]}'
```
(Header names/versions can change — see the official API reference. SDKs set these for you; prefer the SDK in application code.)

## 3. Parameters: what changed and why it matters
- **Sampling params (`temperature`, `top_p`, `top_k`)**: you learned these in Phase 3 — and they still govern *open-weight and many other providers' models*. **But the Claude API has been removing them:** on current Claude 5.x models, non-default values return a 400 error, and per Anthropic's docs a request sending them returns a 400 on **every** model from 2026-10-21. Control behavior with **prompting, `effort`, and structured outputs** instead. (Other providers' reasoning models also restrict sampling params — read each model's docs.) This is a classic "tutorial code from 2024 breaks in 2026" example.
- **Thinking / reasoning**: current Claude models use **adaptive thinking** — the model decides how much to think; you steer with `output_config={"effort": "low|medium|high|xhigh|max"}`. The old `thinking={"type":"enabled","budget_tokens":N}` is rejected on recent models. On the newest Opus-tier model, thinking is always on and `effort` defaults to `medium` — set it explicitly. By default thinking text is **omitted** from responses (empty) unless you request `display: "summarized"`; you are still billed for thinking tokens as output.
```python
client.messages.create(
    model="claude-opus-5-5", max_tokens=16000,
    thinking={"type": "adaptive", "display": "summarized"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "Plan a database migration with zero downtime."}],
)
```
- **No assistant prefill** on recent Claude models (a trick where you started the assistant's reply to force a format). Use **structured outputs** or instructions instead.
- **Forced tool choice** (`tool_choice: any/tool`) returns 400 on the newest Claude models; use `auto` + clear instructions + `strict` tools (see [[03-Structured-Outputs-and-Tool-Use]]).
- Gemini 3-style models expose `thinking_level`; OpenAI reasoning models expose a reasoning-effort control. **Concept is the same: "how long to think" is now a first-class API knob** — and reasoning tokens are billed as output.
- `stop_sequences`, `metadata`, `service_tier`/priority options exist — read the reference.

## 4. Tokens, usage, and cost
- **You pay per token**: input, output, (cached input at a different rate), thinking tokens (as output). Output tokens cost several times input tokens.
- Approximate list prices (cached 2026-11-03, per 1M tokens, input/output — **verify; they change**): Opus 5.5 $4/$20 · Sonnet 5.5 $2/$10 · Haiku 5.5 $0.10/$0.50 (higher beyond 100K prompt) · Fable 5.2 $8/$40. Other providers have comparable tiers.
- **Count tokens before sending** with the provider's tokenizer/endpoint (`client.messages.count_tokens(...)`), not `tiktoken` (different tokenizers).
```python
n = client.messages.count_tokens(model="claude-opus-5-5", messages=messages, system=system).input_tokens
est_cost = n * 4.00 / 1_000_000
```
- Build a tiny **cost meter** now (Exercise A5) — it'll pay for itself.

## 5. Choosing a model tier (first pass; deeper in Note 05)
| Tier | Use for | Notes |
|---|---|---|
| **Flagship** (Opus / GPT top / Gemini Pro) | hard reasoning, agentic coding, ambiguous tasks | quality ceiling; highest cost |
| **Mid** (Sonnet / mini / flash-pro) | production workhorse | best price-performance for most apps |
| **Small** (Haiku / nano / flash-lite) | classification, routing, extraction at scale, sub-agents | cheapest/fastest; set low effort |
| **Frontier-plus** (e.g., Fable-class) | the hardest long-horizon tasks | premium price; stricter data-retention terms may apply |
Process: establish quality ceiling with the best model on **your eval set** → step down while quality holds ([[../Phase-3-Transformers-LLM-Internals/06-Context-Capabilities-and-Limits]] §9).

## 6. Errors, retries, rate limits
Typed exceptions (Python SDK): `BadRequestError` (400, fix your request), `AuthenticationError` (401), `PermissionDeniedError` (403), `NotFoundError` (404 — often wrong model ID), `RateLimitError` (429 — read `retry-after`), `APIStatusError` (≥500 server error → retry), `APIConnectionError`, `APITimeoutError`.
```python
try:
    r = client.messages.create(...)
except anthropic.RateLimitError as e:
    wait = int(e.response.headers.get("retry-after", "60"))
except anthropic.APIStatusError as e:
    if e.status_code >= 500: ...   # retryable
    else: raise                    # 4xx: don't retry blindly
except anthropic.APIConnectionError: ...
```
- **SDKs retry automatically** (connection errors, 408/409/429/5xx; default 2 retries, exponential backoff). Tune with `max_retries`, `timeout`. Write your own retry only for behavior beyond that.
- **Rate limits** are typically **requests/min + input tokens/min + output tokens/min** per model/tier; response headers report remaining quota. New accounts start low; limits rise with usage tier.
- **HTTP 529/overloaded** can occur at peak → backoff + fallback model ([[04-Streaming-Async-Batch-Reliability]]).
- **Timeouts:** default 10 min in the Anthropic Python SDK; set explicit, smaller timeouts in user-facing paths.
- Log `_request_id` with every failure.

## 7. Authentication & secrets (revisit Phase 1)
Environment variables / secret manager; **never commit keys**; separate dev/prod keys; per-environment spend limits; rotate; least privilege; for enterprise: cloud-native IAM (AWS Bedrock/GCP Vertex/Azure Foundry roles), workload identity federation, no long-lived keys on laptops. **Never put an LLM API key in a browser/mobile client** — proxy through your backend.

## 8. SDKs & clients
- Official: `anthropic` (Python 1.x — built on `httpx2`; if you import the HTTP library yourself use `import httpx2 as httpx`), `@anthropic-ai/sdk` (TS), `openai`, `google-genai`. Prefer the official SDK in each language over hand-rolled `requests`.
- Async: `AsyncAnthropic`, `AsyncOpenAI` — required for concurrency ([[04-Streaming-Async-Batch-Reliability]]).
- Cloud-hosted variants: Claude on AWS Bedrock / Google Vertex AI / Microsoft Foundry; OpenAI via Azure OpenAI; Gemini via Vertex. Same models, different auth/regions/quotas/feature availability — **feature parity lags on partner platforms**, check availability tables.
- **OpenAI-compatible endpoints** (vLLM, Ollama, LiteLLM, many hosts) let you point the OpenAI SDK at other backends — convenient for local dev and gateways ([[07-Multi-Provider-and-App-Patterns]]).

## 9. Provider landscape at a glance (verify live)
| Provider | Flagship API shape | Notables |
|---|---|---|
| **Anthropic** | Messages API | tool use, structured outputs, prompt caching, batch, compaction/context editing (beta), Files API, citations, server tools (web search/fetch, code exec), MCP connector, adaptive thinking + effort |
| **OpenAI** | **Responses API** (newer primary) + Chat Completions (legacy-but-common) | function calling with `strict`, Structured Outputs (JSON Schema), built-in tools, reasoning items reusable across calls, separate `refusal` field |
| **Google Gemini** | `generateContent` / newer Interactions API (schema migration in 2026 — check docs) | `thinking_level` (Gemini 3), structured output combinable with built-in tools (search, URL context, code exec), function-calling + built-in tools in one call, context caching, Batch API with webhooks |
| **Open-weight hosts** | OpenAI-compatible | Together, Fireworks, Groq, Cerebras, Bedrock, Vertex, Azure, local vLLM/Ollama |

## 10. Conversation state patterns
1. **Client-managed history (default):** append every user/assistant message (and tool blocks!) and resend. Preserve the *whole* `response.content` (not just text) — it may contain thinking, tool_use, or compaction blocks the API needs back.
2. **Server-assisted state:** OpenAI Responses supports chained responses; Anthropic offers beta *message threads* (smaller requests/lower TTFT in long tool loops, best-effort, keep your own copy) and *compaction* (server summarizes old context; beta). Treat as optimizations; keep a fallback.
3. **Application memory:** your DB stores conversation + user facts; retrieve what's relevant per turn ([[05-Context-Engineering-Caching-Cost]]).
Truncation strategies when history grows: sliding window, summarization/compaction, retrieval of relevant turns, dropping old tool results (context editing).

## 11. A minimal production-grade wrapper
```python
import logging, time, anthropic
log = logging.getLogger(__name__)
client = anthropic.Anthropic(max_retries=3, timeout=60.0)

def ask(messages, *, system=None, model="claude-haiku-5-5", max_tokens=2000, **kw):
    t0 = time.perf_counter()
    r = client.messages.create(model=model, max_tokens=max_tokens, system=system, messages=messages, **kw)
    dt = time.perf_counter() - t0
    if r.stop_reason == "max_tokens": log.warning("truncated id=%s", r._request_id)
    if r.stop_reason == "refusal": log.warning("refusal %s", getattr(r, "stop_details", None))
    log.info("model=%s in=%d out=%d cache_r=%s s=%.2f id=%s", model, r.usage.input_tokens, r.usage.output_tokens,
             getattr(r.usage, "cache_read_input_tokens", 0), dt, r._request_id)
    return "".join(b.text for b in r.content if b.type == "text"), r
```
This is the seed of an **LLM gateway/observability layer** (Phase 7).

## 12. Common beginner mistakes
Assuming `content[0].text` · ignoring `stop_reason` · sending temperature to models that reject it · not logging usage · unbounded history growth · hard-coding model strings everywhere (centralize in config) · pasting keys in code · no timeouts · retrying non-idempotent side effects (tool calls!) · treating the model as deterministic · testing with one example.

## Self-check
1. Why is the API stateless and what does that imply for cost? 2. List five fields you should log per call. 3. What does `stop_reason: "max_tokens"` mean for downstream JSON parsing? 4. Why shouldn't you use tiktoken to estimate Claude costs? 5. What replaced `budget_tokens`? 6. Where must API keys live?

Next → [[02-Prompt-Engineering]]
