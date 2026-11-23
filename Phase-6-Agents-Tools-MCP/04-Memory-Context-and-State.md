---
tags: [phase-6, memory, context, state, durable-execution]
---
# 04 — Memory, Context & State for Agents

> Agents run for dozens to thousands of steps. The binding constraint is **context** (finite, rot-prone — [[../Phase-4-Building-with-LLM-APIs/05-Context-Engineering-Caching-Cost]]) and **continuity** (crashes, multiple sessions, multiple agents). This note is the toolbox.

## 1. Kinds of memory
| Type | Scope | Where it lives | Example |
|---|---|---|---|
| **Working memory** | current step | the context window | the current file, the last tool result |
| **Short-term / episodic (session)** | current task/session | transcript, scratchpad, todo list | plan, decisions, what's been tried |
| **Long-term semantic** | across sessions | files/DB/vector store/memory tool | user preferences, project conventions, learned facts |
| **Long-term episodic** | across sessions | logs/summaries of past runs | "last time deploying to staging failed because…" |
| **Procedural** | across sessions | skills, prompts, playbooks | how to run the quarterly report |
Design question for each piece of information: **where should it live, when is it loaded, who writes it, how does it expire?**

## 2. Within-session context management
| Technique | Mechanism | Notes |
|---|---|---|
| **Stable prefix + caching** | static system/tools first | big cost/latency win |
| **Clear stale tool results** (context editing) | drop old tool outputs/thinking while keeping the call record | cheap, deterministic; invalidates cache after the edit point |
| **Compaction/summarization** | summarize history near the limit, restart with summary + recent turns | lossy; **keep decisions, constraints, open questions, file paths, errors seen; test fact survival** |
| **Offload to files** | write big outputs/plans/notes to disk; keep pointers | agent re-reads slices as needed |
| **Just-in-time retrieval** | store IDs/paths/URLs; fetch on demand | avoids preloading |
| **Todo/plan file** | agent maintains and re-reads a checklist | keeps long tasks on track (recitation focuses attention) |
| **Sub-agents** | isolate noisy exploration in a fresh context; return a summary | [[05-Multi-Agent-Systems]] |
| **Structured note-taking** | agent writes `NOTES.md`/progress logs | survives compaction and resets |
Anthropic's platform offers server-side **compaction** (beta; must pass back the full response content including compaction blocks), **context editing** (beta; clear tool results/thinking), the **memory tool** (client-implemented storage), and **message threads** (beta; smaller requests in long loops) — see docs for current availability and constraints. Preserve **thinking blocks** and any required block types as the API demands when replaying history.

## 3. Cross-session memory patterns
### 3.1 File-based memory (simple & robust)
A `/memories` (or `AGENTS.md`/`CLAUDE.md`-style) directory the agent reads at start and updates when it learns something durable. Pros: transparent, human-editable, versionable, no infra. Cons: needs hygiene as it grows (the index must stay small; use topic files + an index).
```
memories/
  index.md              # short table of contents loaded every session
  user_preferences.md
  project_conventions.md
  gotchas.md
```
**Protocol in the system prompt:** "At the start, read index.md; store durable facts (preferences, decisions, corrections) concisely; update or delete outdated notes; never store secrets."
### 3.2 Database / key-value memory
Structured facts with schema (`user_id, key, value, source, updated_at, confidence`) — good for profiles, entities, CRM-like memory; enables TTL, audit, deletion.
### 3.3 Vector/semantic memory
Embed past interactions/notes; retrieve top-k relevant at each turn (RAG over memory). Good for large histories; risk of retrieving stale/irrelevant items and **memory poisoning**.
### 3.4 Hybrid & hierarchical (MemGPT/Letta-style)
Main context ("RAM") + external archival storage ("disk") with the agent managing paging via tools; core memory blocks always in context, archival searched on demand.
### 3.5 Reflection/summary memory
After a task, the agent writes a lessons-learned summary (Reflexion-style); future runs retrieve relevant lessons.
Frameworks/products: Letta, Mem0, Zep, LangGraph store/checkpointers, LlamaIndex memory modules, provider memory features (e.g., managed memory stores in agent platforms).

## 4. Memory hygiene & risks
- **Write policy:** what deserves saving (stable, reusable, non-sensitive); **who** can write (agent only? user-approved?); require provenance (source/turn/time).
- **Staleness & conflicts:** timestamps, supersession ("user moved to Berlin"), periodic consolidation, conflict resolution, demotion/deletion. Otherwise memory becomes a landfill of outdated beliefs.
- **Privacy & compliance:** PII minimization, user visibility/edit/delete ("forget me"), retention limits, tenant isolation, encryption.
- **Security — memory & context poisoning:** an injected instruction saved into memory persists across sessions (OWASP agentic Top 10 treats this as its own category). Mitigate: don't write untrusted content verbatim; sanitize/summarize; separate "facts" from "instructions"; require confirmation for writes derived from untrusted sources; review memory diffs; scope memory per user/tenant; allow rollback.
- **Evaluation:** test recall after N unrelated turns, update correctness, conflict handling, deletion, poisoning resistance.

## 5. State, checkpointing & durable execution
Long-running agents must survive crashes, deploys, and human delays (approvals can take hours).
- **Explicit state object:** messages + plan + artifacts + step counter + budget used + pending approvals — serializable (Pydantic).
- **Checkpoint after each step** (DB/object store). On restart, **resume** from the last checkpoint; make tool calls **idempotent** (idempotency keys; check "already done?" before redoing) — otherwise retries duplicate emails/payments.
- **Durable execution engines:** Temporal, Inngest, Restate, AWS Step Functions, Azure Durable Functions, Cloudflare Workflows; agent frameworks with checkpointers (LangGraph persistence/interrupts, Pydantic AI durable execution integrations). They handle retries, timers, human-wait states, and replay.
- **Human-in-the-loop as a state:** pause at an approval node, persist, resume on webhook/event. Never block a process thread waiting for a human.
- **Time travel & replay:** with checkpoints/traces you can rewind to a step, edit state, and re-run — invaluable for debugging.
- **Concurrency control:** one active run per entity; locks/leases; avoid two agents editing the same record.
- **Queues & scheduling:** triggers (webhooks, cron, events); rate limits; backpressure; dead-letter queue for failed runs; budgets per run/tenant.

## 6. Long-running agent harnesses (multi-context-window tasks)
For tasks spanning many context windows (large builds/migrations), practitioners' patterns:
1. **Initializer step:** set up the environment, write a **feature/task list** (JSON/markdown with pass/fail status), an init script, and a progress log; commit to git.
2. **Incremental progress:** each session picks **one** unfinished item, implements it, **verifies** (tests/end-to-end checks), updates the task list/progress notes, commits with a clear message.
3. **Clean handoff state:** every session ends in a mergeable, documented state so the next fresh context can orient quickly (`pwd`, read progress file, read git log, run smoke test).
4. **Guardrails against premature "done":** structured task list with explicit statuses; require evidence of verification; forbid deleting/editing tests to pass.
5. **Version control as memory & undo** (git commits, branches).
Why it works: externalizes state to durable artifacts, keeps each context small, and makes progress verifiable.

## 7. Cost/latency interplay
Caching depends on a **stable prefix**: mutating the system prompt or tool list mid-run busts it — use mid-conversation system messages/reminders and append-only history where the platform supports them; avoid reordering earlier messages; remember compaction/editing change the prefix (one-time cache miss). Cost per task = f(loop count, context size, cache hits).

## 8. Design checklist
- [ ] What must persist across sessions, and in what store?
- [ ] What is loaded every session vs on demand (index vs full memory)?
- [ ] Compaction trigger and what's preserved; tests that facts survive.
- [ ] Checkpoint frequency, idempotent tools, resume procedure.
- [ ] Memory write/approval policy; poisoning defenses; user controls.
- [ ] Budgets: tokens/steps/time/$ per run and per tenant.
- [ ] Observability of memory reads/writes.

## Self-check
1. Name four memory types and where each lives. 2. How do compaction, context editing, and memory differ? 3. How would you make a 200-step agent crash-safe? 4. What is memory poisoning and how do you mitigate it? 5. Describe the initializer + incremental-progress harness. 6. Why can compaction break prompt caching, and how do you limit damage?

Next → [[05-Multi-Agent-Systems]]
