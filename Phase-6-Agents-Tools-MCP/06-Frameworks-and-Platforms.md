---
tags: [phase-6, frameworks, platforms]
---
# 06 — Agent Frameworks & Platforms

> **Learn the raw loop first** (Note 09), then adopt a framework *deliberately*. Frameworks accelerate prototypes and give persistence/tracing/handoffs — but add abstraction layers that obscure prompts, change fast, and can hide cost and bugs. 2026 comparison articles agree on a rough split; versions and details vary, so verify before committing.

## 1. Landscape (as described by late-2026 sources — verify)
| Option | Positioning | Strengths | Watch-outs |
|---|---|---|---|
| **Raw SDK + your loop** (Anthropic/OpenAI/Gemini SDKs) | maximum control | transparent, minimal deps, easy to debug, best for learning | you build persistence/tracing/HITL yourself |
| **Provider tool runners** (e.g., Anthropic SDK beta tool runner) | loop helper for *your* tools | tiny; typed tools → schemas; per-turn hooks | not a full agent framework; no built-in tools |
| **Claude Agent SDK** | Claude Code's harness as a library | built-in file/bash/search/web tools, context management, hooks, permissions, subagents, sessions, MCP; strong for coding/filesystem agents | harness-only (you host it); Anthropic-model-centric; read docs for current API |
| **Claude Managed Agents** (beta) | Anthropic hosts loop **and** a per-session sandbox | persisted versioned agent configs, environments, event streaming, vaults (credentials), memory stores, outcomes (graded iteration), multiagent/dynamic workflows, scheduled runs, budgets | beta; extra runtime billing; session history retained; not for ZDR/HIPAA per docs reviewed; vendor lock-in |
| **OpenAI Agents SDK** | lightweight, handoff-based agents | agents + tools + **handoffs** + guardrails + sessions + tracing; simple mental model; works with other models via adapters | sources note it's capable but still pre-1.0 (as of the comparison I saw); complex state machines need more |
| **LangGraph** | agents as **state graphs** | explicit nodes/edges/state, **persistence/checkpointing, interrupts (human-in-the-loop), time-travel**, subgraphs, streaming; model-agnostic; mature ecosystem (LangSmith) | steeper learning curve; more boilerplate; abstraction churn in the wider LangChain family |
| **Google ADK** | Google Cloud/Gemini-oriented | multi-agent hierarchies, A2A + MCP, built-in eval, multi-language, Vertex deployment | strongest fit inside Google stack |
| **Pydantic AI** | type-safe Python agents | Pydantic validation, structured outputs, dependency injection, durable-execution integrations, observability (Logfire) | Python-centric; younger ecosystem |
| **CrewAI** | role-based "crews" | fast prototypes, readable role/task config | teams often outgrow it for complex control flow |
| **AutoGen/AG2, MetaGPT, Semantic Kernel, LlamaIndex agents, Haystack agents, Mastra (TS), Vercel AI SDK (TS), DSPy** | various | niche fits (conversation patterns, enterprise .NET, TS apps, prompt optimization) | evaluate by need |
| **Workflow/durable engines** (Temporal, Inngest, Restate, Prefect, Airflow) | reliability layer | retries, timers, long waits, exactly-once-ish semantics | not LLM-specific; pair with any of the above |
| **Low-code** (n8n, Zapier, Make, Dify, Flowise, Langflow, Copilot Studio, Agentforce, etc.) | citizen-dev/ops automation | speed, connectors | limited control/testing; fine for simple workflows |
Consensus pattern in sources: **LangGraph** for complex/stateful production flows; **vendor SDKs** when committed to one provider; **Pydantic AI** for typed Python services; **CrewAI** for quick role-based prototypes.

## 2. Choosing — decision guide
1. **Is it really an agent?** If a workflow (chain/route/parallel) suffices, a few functions + a durable engine may beat any framework.
2. **Control vs speed:** need to inspect/modify every prompt & state transition → raw loop or LangGraph. Need a working demo today → vendor SDK/CrewAI/low-code.
3. **Hosting model:** Do you want to run the loop (SDK/LangGraph on your infra) or have the vendor run it (Managed Agents)? Compliance (VPC, ZDR, residency) often decides.
4. **Persistence & HITL requirements:** long waits, approvals, resume after crash → checkpointers/durable engines.
5. **Multi-provider needs:** model-agnostic frameworks vs vendor-locked SDKs.
6. **Observability:** native tracing (OpenTelemetry, Langfuse, LangSmith, Logfire, Phoenix).
7. **Team skills & language:** Python vs TypeScript vs .NET/Java.
8. **Maturity & churn:** check release cadence, breaking changes, community; pin versions.
9. **Escape hatch:** can you drop to raw API calls when the abstraction leaks?

## 3. What a framework should give you (checklist)
Tool abstraction (schema from types) · loop with limits · parallel tool execution · streaming · structured outputs · **state persistence & resume** · **interrupts/approvals** · **tracing** · guardrail hooks · sub-agents/handoffs · evals integration · MCP client support · async · testing/mocking utilities · cost/usage accounting.

## 4. Minimal patterns in each style (concepts — consult docs for current syntax)
**Raw loop:** see [[09-Build-an-Agent-From-Scratch]].
**LangGraph-style:** define `State` (TypedDict/Pydantic) → nodes (functions: `call_model`, `run_tools`) → conditional edge (`has_tool_calls? → tools : END`) → compile with a **checkpointer** → invoke with a `thread_id`; use **interrupt** before sensitive nodes to await human approval, then resume.
**OpenAI Agents SDK-style:** `Agent(name, instructions, tools, handoffs, guardrails)` → `Runner.run(agent, input)`; triage agent hands off to specialists; tracing built in.
**Claude Agent SDK-style:** `query(prompt, options)` with allowed tools, permission mode, hooks, subagents, MCP servers; the SDK runs the Claude Code loop and streams messages.
**Managed Agents-style:** create the **agent** once (model, system, tools, MCP servers, skills) as versioned config; create an **environment** (container image/network rules); start **sessions** referencing the agent; stream events; send user messages/tool results/confirmations; store credentials in **vaults**; define **outcomes** with rubrics for deliverables; attach **memory stores**; set **budgets**. (Create agents in setup, not on each request.)

## 5. Protocols & interoperability
- **MCP** — agent↔tools/data ([[03-MCP-Model-Context-Protocol]]).
- **A2A (Agent2Agent)** — agent↔agent across vendors (capability discovery via "agent cards," task lifecycle); complements MCP.
- **AG-UI / generative-UI protocols** — agent↔frontend event streaming (messages, tool calls, state sync) for rich apps.
- **OpenTelemetry GenAI conventions** — standardized tracing of LLM/agent spans.
- **OpenAI-compatible chat/Responses endpoints** — de facto model API interop.
- **Agent Skills format** — portable procedural knowledge folders.
Interop reduces lock-in, but implementations lag specs; test real clients.

## 6. Build vs buy (platforms & products)
Off-the-shelf agent products (coding agents, support agents, research agents, enterprise search/assistants such as Copilot/Glean-style, vertical agent startups) vs custom. **FDE lens:** customers often need the *last mile* — integration with their systems, data permissions, workflows, evals on their cases. Criteria: differentiation, data sensitivity, customization depth, integration surface, total cost, vendor stability, security review burden, time to value. Often best: **buy the platform/runtime, build tools + evals + workflows on top.**

## 7. Migration & portability tips
- Keep **prompts, tool schemas, and evals** framework-independent (files/YAML + pure functions).
- Isolate framework code behind a thin interface (`run_task(task) -> Result`).
- Maintain a **golden task suite** so you can port frameworks/models and prove parity.
- Pin versions; read changelogs; avoid deep reliance on experimental APIs.

## 8. Exercise-style comparison (do this)
Implement the same agent (file-search + calculator + web fetch, 20 test tasks) in: (1) raw loop, (2) LangGraph, (3) a vendor SDK. Compare: lines of code, time to build, debuggability, tracing quality, resume-after-crash, HITL support, tokens/cost, success rate (pass^k), and how easily you can swap models. Write a 1-page recommendation.

## Self-check
1. When is a framework unnecessary? 2. Contrast LangGraph with a vendor SDK on control, persistence, and lock-in. 3. What does "harness-only" mean vs "harness + deployment"? 4. Place MCP, A2A, and AG-UI on an architecture diagram. 5. What artifacts keep you portable across frameworks? 6. Build-vs-buy criteria for an enterprise support agent.

Next → [[07-Agent-Reliability-Safety-and-Security]]
