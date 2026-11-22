---
tags: [phase-4, context-engineering, caching, cost]
---
# 05 — Context Engineering, Prompt Caching & Cost

> **Context engineering** = deciding *exactly what tokens the model sees on each call*. Prompt engineering is a subset (the instructions); context engineering also covers retrieved docs, tool definitions & results, history, memory, and state. In 2025–26 it became the central discipline of agent building, because of **context rot**: as context grows, recall and reasoning degrade ([[../Phase-3-Transformers-LLM-Internals/06-Context-Capabilities-and-Limits]]).

## 1. Principles
1. **Context is a finite, precious budget** — attention is spread over everything; irrelevant tokens (distractors) hurt accuracy, cost, and latency.
2. **Smallest set of high-signal tokens** that makes the desired behavior likely.
3. **Stable first, volatile last** — maximizes cache hits ([[#3. Prompt caching]]).
4. **Just-in-time over eager loading** — keep lightweight references (IDs, file paths, URLs) and let the model fetch details via tools when needed, instead of pre-stuffing everything.
5. **Isolate** — delegate noisy sub-tasks to sub-agents/separate calls and bring back only a summary.
6. **Persist outside context** — notes/memory files/DB; reload selectively.
7. **Measure** — evaluate with *your* data at realistic lengths; test whether facts survive compaction.

## 2. The four context operations (practical taxonomy)
| Operation | What | Use when |
|---|---|---|
| **Load eagerly** | put it in the prompt up front | small, always-needed, stable (system prompt, schema, key policies) |
| **Fetch just-in-time** | tools/retrieval at the moment of need | large or rarely needed info (docs, DB rows, files) |
| **Compact** | summarize/clear older context, keep what still matters | long conversations/agent runs approaching limits |
| **Hand to a sub-agent** | separate context window; return a distilled result | exploratory/noisy subtasks (search, code reading, research) |

### What fills a context window in real apps
System prompt · tool definitions (can be thousands of tokens each set) · conversation history · **tool results (often the biggest culprit)** · retrieved documents · images/PDFs (token-heavy) · reasoning tokens · memory.

## 3. Prompt caching
**Idea:** the provider stores the processed KV state for a *prefix* of your prompt ([[../Phase-3-Transformers-LLM-Internals/05-Inference-and-Decoding]] §2); later requests sharing that **exact prefix** skip recomputation → **much cheaper (cache reads ≈ 10% of input price on Anthropic) and faster TTFT**. Writes cost a bit more (≈1.25× for the default 5-minute TTL on Anthropic; longer TTLs like 1 hour available at a different rate — verify current multipliers).

**Rules (Anthropic semantics; others similar):**
- **Prefix match** — any byte change anywhere in the prefix invalidates everything after it. Render order: `tools` → `system` → `messages`.
- Put **stable content first** (tool defs, system prompt, reference docs, few-shot examples); **volatile content last** (timestamps, user IDs, the question).
- **Breakpoints:** up to 4 `cache_control` markers, or use **automatic top-level caching** which caches the last cacheable block.
- **Minimum cacheable prefix** is model-dependent (hundreds to a few thousand tokens); shorter prefixes silently won't cache.
- **TTL:** default 5 min (refreshed on each hit); optional 1 h.
- **Caches are per-model** (and per workspace/org rules); switching models, changing tools, changing effort/top-level settings can invalidate.
```python
resp = client.messages.create(
    model="claude-opus-5-5", max_tokens=16000,
    cache_control={"type": "ephemeral"},                     # automatic caching of the last cacheable block
    system=LONG_STABLE_SYSTEM_PROMPT_WITH_POLICIES,
    messages=[{"role": "user", "content": question}],
)
u = resp.usage
print(u.cache_creation_input_tokens, u.cache_read_input_tokens, u.input_tokens)   # VERIFY the cache is working
```
**Verify with `usage`**: if `cache_read_input_tokens` stays 0 across repeated requests, a **silent invalidator** is at work: `datetime.now()`/UUID/request ID in the system prompt, unsorted `json.dumps` of dicts, tool list changing order, per-user content before the stable part, different model/params, or TTL expiry.
**Multi-turn:** cache the growing history by moving the breakpoint forward each turn (auto-caching handles this) → each turn only pays full price for the *new* tokens.
**Economics example:** 30k-token policy prefix, 100 requests/hour on a $4/MTok-input model: uncached = 100×30k×$4/1M = **$12/h**; cached ≈ 1 write ($0.15) + 99 reads ($0.012 each ≈ $1.19) ≈ **$1.34/h** (~89% cheaper) — *plus* lower latency. Always compute with current prices.
**Other providers:** OpenAI applies automatic prefix caching on long prompts (with usage fields reporting cached tokens); Gemini offers implicit and explicit context caching. Same design principle: stable prefix first.
**Pre-warming** and **mid-conversation system messages** (append operator instructions without editing the cached prefix) are advanced techniques on supporting models.

## 4. Managing long conversations & agent runs
| Technique | How | Trade-off |
|---|---|---|
| **Sliding window** | keep last N turns | loses early facts |
| **Summarization** | LLM summarizes old turns | lossy; costs a call; can drop crucial details |
| **Server-side compaction (Anthropic beta)** | API auto-summarizes when input nears a trigger (default ~150K); returns a `compaction` block you must pass back | easiest; **append `response.content`, not just text** |
| **Context editing (beta)** | automatically **clear** old tool results/inputs or thinking blocks (`clear_tool_uses_20250919`, `clear_thinking_20251015`) | cheap, deterministic; not a summary |
| **Memory tool / files** | model reads/writes notes to external storage (`memory_20250818` tool, or your own `NOTES.md`) | durable across sessions; needs hygiene (dedupe/expire/conflict resolution) |
| **Retrieval of history** | embed & fetch relevant past turns | complexity |
| **Message threads (beta)** | provider keeps recent conversation; you send only new messages | latency/size win; best-effort — keep your own copy; can't combine with compaction |
```python
# Server-side compaction (beta) — preserve the full content
response = client.beta.messages.create(
    betas=["compact-2026-01-12"], model="claude-opus-5-5", max_tokens=16000,
    messages=messages, context_management={"edits": [{"type": "compact_20260112"}]})
messages.append({"role": "assistant", "content": response.content})   # NOT just the text
```
**Compaction design rules:** relocate details to files/memory and leave pointers; keep decisions, open questions, constraints, user preferences, and recent turns verbatim; **test** that specific facts survive (probe questions after compaction); don't delete cache-critical early content casually (invalidates caches).

## 5. Tool-result bloat (the #1 hidden cost)
- Return **concise** results; paginate; offer `fields=`/`limit=`; summarize server-side; strip HTML/boilerplate; truncate with an explicit note ("showing 10 of 240; call again with `offset`").
- Clear stale tool results via context editing once they've been used.
- Prefer **programmatic tool calling / code execution** to process large data in a sandbox and return only the answer.
- Tool **definitions** also cost tokens every call → keep the tool set small, or use tool search/deferred loading for large catalogs.

## 6. Token budgeting & counting
- **Count before you send** for large/uncertain inputs: `client.messages.count_tokens(...)` (provider-specific tokenizers; images/PDFs count too).
- Budget table per feature: system, tools, history, retrieval, user input, **reserved output**, thinking. Enforce caps in code (truncate/summarize/error) and **never silently truncate** user content — tell the user or chunk.
- **Task budgets (Anthropic beta):** give an agentic loop a token ceiling the model can pace itself against (distinct from `max_tokens`).
- Re-baseline token counts when changing models (tokenizers differ across generations).

## 7. Cost modeling (do this for every feature)
```
cost/request = in_uncached·p_in + cache_write·p_in·1.25 + cache_read·p_in·0.10 + out(+thinking)·p_out
cost/task    = Σ requests per completed task (retries + tool loops + judge calls)
monthly      = cost/task × tasks/month × (1 + failure/retry overhead)
```
- Output tokens ≫ input price; **reasoning tokens are output**.
- Judge **cost per completed task**, not per request (a cheap model needing 3 retries may cost more).
- **Levers, roughly in order of "free" to "tradeoff":** (1) prompt caching, (2) trim input tokens (dedupe, compress, concise tool results), (3) loop hygiene (max steps, stop early), (4) shorter outputs (instruct length, structured outputs), (5) **Batch API (−50%)** for offline, (6) lower **effort**/thinking budget per route, (7) smaller model for easy routes (routing/cascades — measured against evals), (8) fine-tune/distill for very high volume (Phase 7).
- Before building a multi-model cascade, test **the strongest model at lower effort** on the same tasks — often matches prior-gen quality at lower cost and keeps one cache namespace.
- Use provider **usage/cost APIs & dashboards**; tag requests with feature/tenant; alert on anomalies; set hard spend limits.

## 8. Latency modeling
`latency ≈ network + queue + TTFT(input tokens, cache hit?) + output_tokens × TPOT (+ tool round trips)`
Levers: **cache prefixes** (TTFT), **stream** (perceived), **shorter outputs**, **parallelize independent calls**, **smaller model/lower effort** for simple steps, **fast mode** (premium price, higher tok/s, limited models), **reduce tool round trips** (batch tools, programmatic tool calling), **colocate** with provider region, **speculative UX** (start rendering/prefetch). Measure p50/p95/p99, not averages.

## 9. Context-engineering checklist for a new feature
- [ ] Which tokens are *stable* vs *volatile*? Order accordingly; place cache breakpoint.
- [ ] What's the max context at p99? What happens when exceeded?
- [ ] Which info can be fetched JIT instead of preloaded?
- [ ] Are tool results bounded? Stale ones cleared?
- [ ] Do we log token counts by section (system/tools/history/retrieval/output)?
- [ ] Is there a compaction/memory strategy + tests that facts survive?
- [ ] Do we have a cost/task budget & alert?
- [ ] Have we evaluated at realistic long-context lengths?

## Self-check
1. Define context engineering vs prompt engineering. 2. What invalidates a prompt cache? List five. 3. Compute savings for a 20k-token prefix over 500 calls (state prices). 4. Why is "append `response.content`, not text" critical with compaction? 5. Name three ways to shrink tool-result bloat. 6. Why judge cost per completed task?

Next → [[06-Multimodal-and-Documents]]
