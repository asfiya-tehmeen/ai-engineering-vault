---
tags: [phase-4, structured-outputs, tool-use, function-calling]
---
# 03 — Structured Outputs & Tool Use

> These two features turn a text generator into a **component in a software system**: structured outputs make the model's *answers* machine-readable; tool use lets the model request *actions* from your code. Agents (Phase 6) are loops of tool calls.

## Part A — Structured outputs

### Why
Free-text → regex → brittle. Instead **constrain decoding to a JSON Schema** (the grammar-masked decoding from [[../Phase-3-Transformers-LLM-Internals/05-Inference-and-Decoding]] §4). Result: syntactically valid JSON that matches your schema. Guarantees *shape*, not *truth*.

### Pydantic + `parse` (Anthropic Python SDK)
```python
from pydantic import BaseModel
from typing import List
import anthropic

class Contact(BaseModel):
    name: str
    email: str
    plan: str
    interests: List[str]
    demo_requested: bool

client = anthropic.Anthropic()
resp = client.messages.parse(
    model="claude-opus-5-5", max_tokens=16000,
    messages=[{"role": "user", "content": "Extract: Jane Doe (jane@co.com) wants Enterprise, interested in API and SDKs, wants a demo."}],
    output_format=Contact,
)
contact = resp.parsed_output          # validated Contact instance
```
(The helper's keyword is `output_format=` on `parse`; on raw `create` calls the schema goes in `output_config={"format": {...}}` — the older top-level `output_format` on `create` is deprecated. SDK signatures evolve; check the current reference.)

### Raw JSON Schema
```python
resp = client.messages.create(
    model="claude-opus-5-5", max_tokens=16000,
    messages=[{"role": "user", "content": "Extract info: John Smith (john@example.com) wants the Enterprise plan."}],
    output_config={"format": {"type": "json_schema", "schema": {
        "type": "object",
        "properties": {"name": {"type": "string"}, "email": {"type": "string"}, "plan": {"type": "string"}},
        "required": ["name", "email", "plan"], "additionalProperties": False}}},
)
import json
data = json.loads(next(b.text for b in resp.content if b.type == "text"))
```
Notes: object schemas need `additionalProperties: false` + `required`; providers support a **subset** of JSON Schema (limited recursion/format/constraints — read the docs; Pydantic helpers often transform schemas for you). Structured output may be incompatible with some features (e.g., **citations** on Claude return a 400) — check combos.

### Other providers
OpenAI: Structured Outputs with `strict: true` (JSON Schema), SDK `parse` helper for Pydantic, separate `refusal` field; **not compatible with parallel function calls** in some modes (set `parallel_tool_calls=false`); schemas aren't Zero-Data-Retention eligible (per docs at time of search). Gemini 3: structured outputs can be **combined with built-in tools** (search, URL context, code execution, function calling). **Always verify current limits.**

### Design good schemas
- **Descriptions are prompts:** add `description` to each field ("ISO-8601 date", "amount in USD cents").
- Use **enums** for closed sets; `null`/optional for missing info — *and say "don't guess"*. Forced-choice fields invite fabrication.
- Order fields so **reasoning/evidence comes before the verdict** (`rationale` → `decision`) — generation is left-to-right, so putting `reasoning` first can improve accuracy.
- Keep schemas flat-ish; limit giant enums; avoid deep nesting.
- Include an `"unknown"`/`"other"`/`"not_found"` option.
- Add a **confidence or evidence quote** field when downstream review is needed.
- Validate *semantics* in code (dates real? totals add up? IDs exist?) — schema validity ≠ correctness.

### Failure handling checklist
1. `stop_reason == "max_tokens"` → output truncated, JSON invalid → raise `max_tokens`/stream/shorten schema.
2. `stop_reason == "refusal"` → handle gracefully (don't parse).
3. Pydantic validation error → log + retry once with the error message in context ("your last output failed validation: …") → fallback/human review.
4. Keep the **raw response** for debugging.
5. Libraries that wrap this pattern across providers: **Instructor**, **Pydantic AI**, **Outlines** (local), LangChain structured output (see resources).

---

## Part B — Tool use (function calling)

### The idea
You describe **tools** (name, description, JSON-Schema input). The model may respond with a **`tool_use` block** — a *request* to call a tool with arguments. **Your code executes it** and returns a **`tool_result`**; the model continues. The model never runs your code; you stay in control (permissions, validation, rate limits).

```
user → model → (tool_use: get_weather{"city":"Paris"}) → YOUR CODE runs it → tool_result → model → final answer
```

### Defining a tool
```python
tools = [{
    "name": "get_order_status",
    "description": "Look up the status of a customer order by order ID. Use when the user asks where their order is. Returns status, carrier, and ETA. Does NOT modify orders.",
    "input_schema": {
        "type": "object",
        "properties": {"order_id": {"type": "string", "description": "Order ID like 'ORD-12345'"}},
        "required": ["order_id"], "additionalProperties": False},
    "strict": True,          # guarantee tool_use.input matches the schema
}]
```

### The manual loop (write this by hand at least once)
```python
messages = [{"role": "user", "content": "Where is order ORD-123?"}]
while True:
    resp = client.messages.create(model="claude-opus-5-5", max_tokens=16000, tools=tools, messages=messages)
    if resp.stop_reason == "end_turn": break
    if resp.stop_reason == "pause_turn":                       # server-side tool paused: resend to continue
        messages.append({"role": "assistant", "content": resp.content}); continue
    if resp.stop_reason in ("max_tokens", "refusal"): raise RuntimeError(resp.stop_reason)

    messages.append({"role": "assistant", "content": resp.content})          # keep ALL blocks
    results = []
    for b in resp.content:
        if b.type == "tool_use":
            try:
                out = run_tool(b.name, b.input)                  # YOUR implementation; validate & authorize!
                results.append({"type": "tool_result", "tool_use_id": b.id, "content": out})
            except Exception as e:
                results.append({"type": "tool_result", "tool_use_id": b.id, "content": f"Error: {e}", "is_error": True})
    messages.append({"role": "user", "content": results})                    # ALL results in ONE user message
print(next(b.text for b in resp.content if b.type == "text"))
```
Rules that bite:
- Return **every** `tool_result` for each `tool_use`, **in a single user message**, with the matching `tool_use_id`. Splitting results across messages can teach the model to stop calling tools in parallel.
- Use `is_error: true` for failures (don't drop the result) and write **actionable error messages** ("Order not found; ask the user to confirm the ID") so the model can recover.
- **Parallel tool use:** one response may contain several `tool_use` blocks; run them concurrently (`asyncio.gather`).
- **Preserve the entire assistant content** (including thinking/tool blocks) when replaying history.
- **Loop limits:** max iterations, token/time budgets, and cost caps — an unbounded agent loop is an unbounded bill.
- **Truncated tool input** (`max_tokens` mid-call) → don't execute partial calls.

### The tool runner (SDK convenience, beta)
```python
from anthropic import beta_tool

@beta_tool
def get_order_status(order_id: str) -> str:
    """Look up the status of a customer order by ID.

    Args:
        order_id: Order ID like 'ORD-12345'.
    """
    return lookup(order_id)

runner = client.beta.messages.tool_runner(
    model="claude-opus-5-5", max_tokens=16000, tools=[get_order_status],
    messages=[{"role": "user", "content": "Where is order ORD-123?"}],
)
for message in runner:
    print(message)
```
Generates the schema from type hints/docstrings and drives the loop. Drop to the manual loop for custom control; both are fine — but understand the manual loop first.

### `tool_choice` today
Default `auto`. **Forcing a call (`any`/specific tool) now returns 400 on the newest Claude models** (older models still accept it). Steer with the prompt ("use the `get_order_status` tool"), keep `strict: true` for schema-valid args, and check that a `tool_use` block actually came back (re-prompt if not). If you only wanted JSON back, use **structured outputs** instead of a fake "return_answer" tool. OpenAI/Gemini retain their own tool-choice modes — read each provider's docs.

### Tool design (this is where agents succeed or fail)
1. **Fewer, higher-level tools** that map to user goals beat dozens of thin API wrappers (e.g., `schedule_meeting` instead of `list_calendars`+`find_free_slots`+`create_event`... unless you need the granularity).
2. **Names & descriptions are prompts.** State *when* to use it, *when not*, what it returns, side effects, and examples. Unambiguous parameter names (`user_email` not `user`).
3. **Return concise, relevant, human-readable results** — not raw 50 KB JSON. Include identifiers needed for follow-up calls. Support pagination/filters/`limit`. Token-efficient responses preserve context.
4. **Helpful errors** that suggest the fix.
5. **Idempotency & safety:** read-only vs write tools; require confirmation for destructive ops; use idempotency keys; scope credentials per user; validate arguments server-side (never trust model-provided paths/IDs — authorize them, confine file paths to a root, parameterize SQL).
6. **Namespacing** (`crm_search`, `docs_search`) when many tools.
7. **Test tools** with evals of real tasks: which tool did it choose, with what args, how many calls?
8. Keep the **tool list stable** and ordered (cache-friendly) — changing tools invalidates cached prefixes.
9. Many tools (dozens+) bloat context → **tool search / deferred tool loading** features let the model discover tools on demand.

### Server-side (provider-run) tools
Run on the provider's infrastructure; you just declare them and receive results in the same response (no execution loop for you):
- **Web search** and **web fetch** (Anthropic: `web_search_20260209`, `web_fetch_20260209` on current models; fetch only fetches URLs already present in the conversation), **code execution** sandbox (`code_execution_*`; can generate files/charts), **memory tool**, **MCP connector** (remote MCP servers via `mcp_servers` + `mcp_toolset`), **tool search**. OpenAI and Gemini offer analogous built-in tools (search, code interpreter, file search, computer use).
- Server-tool *errors* can arrive as HTTP 200 with an error object in the result block — branch on it.
- Governance: allowed/blocked domains, max uses, data-retention implications, cost per search.
- Programmatic tool calling: the model writes code in the sandbox that calls *your* tools, reducing round trips and context bloat (advanced).

### MCP — Model Context Protocol (preview; deep dive in Phase 6)
An open protocol for exposing **tools, resources, prompts** to any LLM app via standardized servers (stdio/HTTP). Write a tool once as an MCP server; reuse across clients. Anthropic's API can connect to remote MCP servers directly; SDK helpers convert MCP tools for the tool runner.

### Security notes (preview of Phase 7)
- Tool outputs are **untrusted input** (web pages, emails, documents) → prompt-injection vector. Never let a tool result silently grant privileges.
- **Least privilege + human-in-the-loop** for irreversible actions; log every tool call with args/results (audit trail).
- Rate-limit and budget tool calls per session.

## Part C — Putting both together
Typical production pattern: **tools for gathering/acting, structured outputs for the final answer.** E.g., support triage: tools (`get_customer`, `get_order`, `search_kb`) → final structured result `{category, urgency, suggested_reply, needs_human}`. Validate with Pydantic; route to human if `needs_human` or validation fails.

## Self-check
1. Who executes a tool call? 2. Why return all tool results in one user message? 3. What does `strict: true` guarantee — and not guarantee? 4. How do you get guaranteed-JSON output now that forced tool choice is gone on newest models? 5. Name five properties of a good tool description. 6. Why must you authorize model-supplied arguments server-side?

Next → [[04-Streaming-Async-Batch-Reliability]]
