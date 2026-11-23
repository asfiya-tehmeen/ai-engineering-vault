---
tags: [phase-6, agents, fundamentals]
---
# 01 — Agent Fundamentals

## 1. What is an agent?
**Workflow:** LLM calls and tools orchestrated by **predefined code paths** (you decide the steps).
**Agent:** the **LLM dynamically directs its own process** — choosing which tools to call, in what order, and when it's done — using environmental feedback (tool results) at each step.
Minimal definition: **an LLM + tools + a loop + a stopping condition**, running against an environment.
```
goal → [ observe context → think/plan → choose action (tool call) → execute → observe result ] → … → done / give up / ask human
```
Simplest rule from practitioners: **start with the simplest solution; add agentic complexity only when it demonstrably pays off.** Agents trade **latency, cost, and predictability** for **flexibility and capability on open-ended tasks**.

## 2. The autonomy spectrum
| Level | Description | Example |
|---|---|---|
| 0 | Single LLM call | classify a ticket |
| 1 | LLM + retrieval/tools in a fixed pipeline | RAG answer |
| 2 | **Workflow** with branching/parallel/loops defined in code | triage → route → draft → review |
| 3 | **Agent** chooses tools/steps; human approves risky actions | coding assistant, research assistant |
| 4 | Agent runs long, autonomous tasks with checkpoints and guardrails | overnight migration/refactor agent |
| 5 | Multi-agent, self-directed, minimal oversight | research/ops "teams" (rare; high risk) |
Pick the **lowest level that meets the requirement.** OWASP's agentic guidance calls this **least agency**: autonomy must be *earned* by a task, not granted by default.

## 3. Should you build an agent? (four checks)
1. **Complexity** — is the task multi-step and hard to specify fully in advance? (If you can draw the flowchart, build a workflow.)
2. **Value** — does the outcome justify higher cost/latency?
3. **Viability** — can the model do each *type* of step reliably (tool use, reading, reasoning)? Prototype manually first.
4. **Cost of error** — can mistakes be caught and recovered (tests, review, undo, sandbox)? If errors are irreversible and costly, add human gates or don't automate.
If any answer is "no," stay simpler.

## 4. Five workflow patterns (know them cold)
(From Anthropic's *Building Effective Agents*; widely adopted vocabulary.)
1. **Prompt chaining** — fixed sequence; each LLM call processes the previous output; add programmatic **gates** between steps. Use when a task decomposes cleanly (outline → check → write).
2. **Routing** — classify input, dispatch to a specialized prompt/model/tool. Use for distinct categories (support intents, difficulty → model size).
3. **Parallelization** — (a) *sectioning*: independent subtasks run concurrently; (b) *voting*: same task multiple times for confidence. Use for speed or diverse perspectives (guardrail checks alongside the main call).
4. **Orchestrator–workers** — an LLM dynamically breaks down a task and delegates subtasks to workers, then synthesizes. Use when subtasks can't be predicted (multi-file code changes, research).
5. **Evaluator–optimizer** — one LLM generates, another critiques against criteria, loop until good. Use when there are clear evaluation criteria and iteration improves quality (translation, code with tests).
Real systems compose these. An **agent** is the generalization where the model, not your code, decides the composition at runtime.

## 5. The agent loop in detail
```python
state = init(goal, context)
for step in range(MAX_STEPS):
    resp = llm(state.messages, tools)                    # model proposes text and/or tool calls
    if resp.stop_reason == "end_turn": return resp       # finished
    for call in resp.tool_calls:                         # (parallel when independent)
        guard(call)                                      # permissions, schema validation, budget, approval
        result = execute(call)                           # sandboxed, timed-out, size-capped
        state.append(result)                             # concise, informative; errors are actionable
    state = maybe_compact(state)                         # context management
raise BudgetExceeded
```
Essential harness responsibilities (the model does *reasoning*; **you** own these):
- **Stop conditions:** natural end, max steps, token/$ budget, wall-clock timeout, repeated-failure detector (same call 3×), "needs human."
- **Tool execution safety:** validation, authorization, sandboxing, timeouts, output size caps, idempotency.
- **Context management:** compaction, clearing stale tool results, memory ([[04-Memory-Context-and-State]]).
- **Observability:** log every step (thought summary, tool, args, result size, latency, tokens).
- **State persistence:** checkpoint so crashes resume instead of restart.
- **Human-in-the-loop hooks:** approvals, questions, interrupts.

## 6. Reasoning patterns
- **ReAct** (Reason + Act): interleave thought → action → observation. Modern reasoning models and adaptive thinking do this natively between tool calls; you don't need to prompt "Thought:/Action:" scaffolds. Thinking text may be summarized/hidden and isn't a faithful audit log — **audit actions and results, not just reasoning.**
- **Plan-and-execute:** make an explicit plan (a todo list), then execute steps, re-planning on surprises. Great for long tasks; keep the plan in a file/state the agent updates (it also anchors attention).
- **Reflection/critique:** after acting, verify (run tests, re-read output, check constraints). **Verification is the single best reliability lever** — design tasks with checkable feedback (tests, linters, schema validation, diffing).
- **Search/tree exploration** (best-of-N, MCTS-like) for hard problems; expensive.
- **Self-consistency / voting** on high-stakes decisions.
- **Effort/thinking controls:** tune per step type (cheap/fast for routing & summarizing; high for planning & debugging).

## 7. What makes tasks agent-friendly
- **Clear, checkable success criteria** (tests pass, schema valid, record created).
- **Rich, reliable tools** with fast, informative feedback.
- **Bounded environment** (sandbox, scoped credentials, reversibility).
- **Decomposable with recoverable steps.**
- **Value in flexibility** (varied inputs, unpredictable paths).
Poor fits: tasks needing deterministic, auditable, identical outputs every time (use workflow/code); irreversible high-stakes actions without review; tasks where you can't observe success.

## 8. Common agent use cases (and typical architecture)
| Use case | Pattern | Notes |
|---|---|---|
| **Coding agent** (fix issues, refactor) | single agent + file/bash/test tools | tests are the verifier; sandbox essential |
| **Research agent** | orchestrator–workers, search/fetch tools | citations; breadth vs cost |
| **Customer support agent** | routing + tools (lookup, refund policy) + guardrails | policy compliance, handoff to human, simulated-user evals (τ-bench style) |
| **Data analyst agent** | SQL/code-execution tools | read-only DB, result validation |
| **Browser/computer-use agent** | screenshot + actions loop | slow/brittle; sandboxed VM; high injection risk |
| **IT/DevOps agent** | CLI tools with approval gates | blast radius control |
| **Document-processing agent** | workflow + exceptions handled by agent | keep deterministic path for the 90% case |
| **Sales/ops automation** | CRM/email tools, approvals | confirm before sending/modifying |

## 9. Cost & latency reality
Agents multiply tokens: each loop re-sends context (mitigated by **prompt caching**), plus tool outputs. Multi-agent systems can use **several×** the tokens of a chat. Design: cache stable prefix, cap steps, clear stale tool results, use smaller models for easy steps/subagents, parallelize independent calls, set per-task budgets, and **measure cost per completed task** ([[../Phase-4-Building-with-LLM-APIs/05-Context-Engineering-Caching-Cost]]).

## 10. Mindset: what changes when you go agentic
- Errors **compound** across steps (95% per-step success → ~60% over 10 steps). Reliability needs verification, retries, and smaller steps.
- **Non-determinism** per run → evaluate over repeated trials (pass^k), not single runs ([[08-Agent-Evaluation-and-Observability]]).
- **Tools are the product surface** — most failures trace to unclear tools/results ([[02-Tool-Design-and-Environments]]).
- **Security expands** — the agent holds credentials and reads untrusted content ([[07-Agent-Reliability-Safety-and-Security]]).
- You're building **harness + environment + evals**, not just prompts.

## Self-check
1. Define agent vs workflow with one example each. 2. Name the five workflow patterns and a use case for each. 3. Apply the four "should I build an agent?" checks to an invoice-approval process. 4. List six responsibilities of the harness. 5. Why can 95% per-step accuracy be a problem? 6. What is the best single reliability lever for agents?

Next → [[02-Tool-Design-and-Environments]]
