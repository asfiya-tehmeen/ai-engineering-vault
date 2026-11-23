---
tags: [phase-6, multi-agent]
---
# 05 — Multi-Agent Systems

> Multi-agent is the most over-hyped and most misused agent pattern. It *does* help in specific situations — mainly **parallelizable breadth** and **context isolation** — and it *always* costs more tokens. Default to a **single agent with good tools**; add agents only when you can name the benefit and measure it.

## 1. Why use multiple agents? (legitimate reasons)
1. **Context isolation** — a sub-agent explores in its *own* fresh window (reading 50 files, searching 100 pages) and returns a **compact summary**, keeping the lead agent's context clean. This is the single best reason.
2. **Parallelism / breadth** — fan out independent subtasks (research N sources, process N documents, review N files) concurrently; latency and coverage improve.
3. **Specialization** — different system prompts/tools/models per role (security reviewer, SQL expert), smaller tool sets per agent (less confusion), cheaper models for easy roles.
4. **Separation of duties / safety** — a "reader" agent that touches untrusted content has **no** write/outbound tools; an "actor" agent with tools never sees raw untrusted text (a key injection-defense pattern); independent **verifier/critic** to check the worker.
5. **Scale beyond one window** — tasks whose total information exceeds any single context.
**Illegitimate reasons:** "it looks cool," role-play teams ("CEO agent, CTO agent") for tasks one agent could do, imitating org charts. Evidence from practice and research (e.g., studies of why multi-agent systems fail) shows frequent coordination failures: specification ambiguity, inter-agent misalignment, duplicated work, premature termination, weak verification.

## 2. Cost reality
Multi-agent systems consume **several times** the tokens of a single agent chat (reported figures for research-style systems are on the order of ~15× a chat and ~4× a single agent — *indicative, vendor-reported; measure yours*). They pay off when task **value** is high and work is **breadth-first and parallelizable**; they're poor for tightly interdependent work (e.g., most coding changes requiring shared state) unless carefully partitioned.

## 3. Core patterns
| Pattern | Structure | Use |
|---|---|---|
| **Orchestrator–workers (lead + subagents)** | lead decomposes, spawns workers in parallel, synthesizes | research, multi-file analysis, batch processing |
| **Handoffs / routing between specialists** | a triage agent transfers control (and conversation) to a specialist | customer support (billing/tech/returns) |
| **Pipeline (sequential roles)** | A → B → C, each transforms | content: research → draft → edit |
| **Evaluator–optimizer / critic loop** | generator + reviewer iterate | code with tests, writing with rubric |
| **Debate / ensemble / voting** | several agents propose; judge or vote | high-stakes decisions, hard reasoning (costly) |
| **Hierarchical teams** | managers of managers | very large tasks (high coordination risk) |
| **Blackboard/shared workspace** | agents read/write a shared store | collaborative artifacts |
| **Dynamic workflows** | the lead writes a *workflow* (phases, fan-out, merge) that a runtime executes with many agents | hundreds of documents/sources in phases |
| **Advisor pattern** | a cheaper executor consults a stronger model mid-task at decision points | cost-aware quality boost |

## 4. Designing a lead–subagent research system (reference design)
1. **Lead agent** analyzes the query, **plans**, saves the plan to memory/state (survives context limits), decides how many subagents and how to split work.
2. **Subagents** (parallel) each get: a clear **objective**, **output format**, **tools/sources guidance**, and **task boundaries** (so they don't duplicate). They search, read, evaluate, and return **condensed findings with sources**.
3. Lead **synthesizes**, may spawn follow-ups, then a **citation/verification** pass checks claims against sources.
4. Effort scaling rules embedded in prompts (simple lookup → 1 agent, few calls; comparison → 2–4 agents; deep research → 10+) to avoid over-spending.
Lessons commonly reported: **teach the orchestrator how to delegate** (vague delegation → duplicated/unfocused work); **start wide then narrow** in searching; let subagents **parallelize tool calls**; **write intermediate outputs to a filesystem/store** rather than routing everything through the lead ("telephone game" loses fidelity); build **evals early** (even 20 queries) and use **LLM-judge rubrics + human review**; invest in **tracing/observability**; use **rainbow deployments** (gradual rollout) for stateful agents; handle errors by letting agents adapt to tool failures gracefully with checkpoints.

## 5. Communication & shared state
- **Message passing** (structured, typed) vs **shared memory/blackboard** (files/DB). Prefer **artifact references** (file paths/IDs) over pasting large content.
- **Contracts between agents:** JSON-schema'd inputs/outputs; explicit success criteria; error semantics.
- **Handoff payloads:** what context transfers (summary vs full history)? Full history is expensive and leaks irrelevant/untrusted content; summary can lose detail — design deliberately.
- **Termination:** who decides "done"? Budget caps per subagent; global deadline; max depth (prevent runaway spawning).
- **Identity & permissions:** each agent has its own scoped credentials/tools (least privilege). A lead shouldn't inherit all worker permissions; workers shouldn't see each other's secrets.
- **Conflict handling:** two agents editing the same file/record → locks, ownership by partition (each agent owns a distinct subset), or merge steps with verification.

## 6. Failure modes (and mitigations)
| Failure | Mitigation |
|---|---|
| Duplicate/overlapping work | explicit task boundaries; shared task board; orchestrator assigns disjoint scopes |
| Vague delegation → wrong output | detailed task descriptions + examples + output schemas |
| Telephone game information loss | artifacts on shared store; return structured evidence with sources |
| Runaway spawning / cost blowup | budgets, max agents/depth, effort-scaling rules, kill switch |
| Infinite loops / ping-pong | step caps, repeated-state detection |
| Cascading errors / bad subagent claims accepted | independent verification, source requirements, spot-check evals |
| Premature termination ("good enough") | success criteria + verifier; completion checklist |
| Prompt-injection spread between agents | treat inter-agent messages from tool-using agents as **untrusted**; separate reader/actor; schema-validate messages |
| Context divergence / inconsistent assumptions | shared state doc; pinned constraints |
| Debugging nightmare | per-agent traces, correlation IDs, replay |

## 7. Platform support (late-2026 landscape — verify)
- **Claude API / Managed Agents:** hosted agent runtime with persisted versioned **agents**, per-session **environments/containers**, **multiagent** rosters (subagents incl. "self" and cheaper worker agents), **dynamic workflows** (agent-written multi-phase fan-out runs), **advisor** consultation, **outcomes** (rubric-graded iterate loop), vaults for credentials, memory stores, scheduled deployments, hard **session budgets**. Beta; runtime billed on top of tokens; not available under zero-data-retention/HIPAA in the docs I reviewed — check eligibility. (Separate product: the **Claude Agent SDK** packages Claude Code's harness as a library with subagents/hooks/permissions.)
- **OpenAI Agents SDK:** agents, **handoffs**, guardrails, sessions, tracing.
- **LangGraph:** graph of nodes (agents/tools) with shared state, supervisors, subgraphs, persistence.
- **Google ADK:** multi-agent hierarchies, A2A and MCP support, built-in eval.
- **CrewAI / AutoGen / MetaGPT / AG2:** role-based teams; popular for prototypes (teams often migrate to more controllable graphs for production).
- **A2A protocol:** cross-vendor agent-to-agent discovery and task delegation (complements MCP).

## 8. When multi-agent wins — decision test
Ask: (1) Is the work **parallelizable** into independent chunks? (2) Would one agent's **context overflow** or rot? (3) Is the task **valuable enough** to pay ~several× tokens? (4) Can I specify **clear contracts** for subtasks? (5) Do I have **evals** to prove it beats a single agent? If you can't say "yes" to most, use a single agent (or a workflow).

## 9. Experiment you should run (to learn the trade-offs)
Same 30 research questions with: (A) single agent; (B) orchestrator + 3 workers; (C) orchestrator + 8 workers. Measure answer quality (rubric), coverage, citations accuracy, tokens, cost, wall-clock, variance. Plot quality vs cost; find where more agents stop helping.

## Self-check
1. Give three legitimate reasons for multi-agent and two bad ones. 2. Why is context isolation the strongest argument? 3. Describe an orchestrator–worker research design and its delegation instructions. 4. List five multi-agent failure modes and mitigations. 5. How can multi-agent design *improve* security? 6. How do you decide with data whether multi-agent is worth it?

Next → [[06-Frameworks-and-Platforms]]
