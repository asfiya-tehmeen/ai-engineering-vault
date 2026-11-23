---
tags: [phase-6, index, agents]
---
# Phase 6 — Agents, Tools & MCP

> [!abstract] Why this phase exists
> An **agent** is an LLM that **decides its own next step** — calling tools, reading results, and looping until a goal is met. Agents turn LLMs from "answer machines" into "workers": coding, research, ops automation, customer service, data analysis. They are also where reliability, cost, and **security** problems get serious. FDEs are often asked: *"Can an agent do this?"* — and the right answer is usually **"here's the simplest thing that works, and here's how we'd measure and contain it."*

**Duration:** 4–6 weeks · **Prereq:** Phase 4 (tool use loop, context engineering), Phase 5 (retrieval as a tool), Phase 1 (async, Docker).

> [!warning] Freshness
> Agent tooling changes faster than anything else in this vault. **Principles** (loops, tools, context, memory, evals, least privilege) are stable; **frameworks, protocol versions, and benchmark leaderboards are not.** Where I cite specifics (MCP spec revision, framework positioning, benchmark status) they come from late-2026 web sources of mixed quality — verify against the official docs in [[12-Resources-and-Practice-Links]] before building on them.

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-Agent-Fundamentals]] | agent vs workflow, the loop, ReAct, five workflow patterns, when to build an agent, autonomy levels |
| 02 | [[02-Tool-Design-and-Environments]] | tool surface (bash vs dedicated), programmatic tool calling, tool search, skills, code execution, computer/browser use |
| 03 | [[03-MCP-Model-Context-Protocol]] | architecture, primitives, transports, auth, building servers, security, MCP vs alternatives, A2A |
| 04 | [[04-Memory-Context-and-State]] | short/long-term memory, files as memory, compaction, checkpointing, durable execution, long-running harnesses |
| 05 | [[05-Multi-Agent-Systems]] | orchestrator–workers, handoffs, subagents for context isolation, costs, failure modes |
| 06 | [[06-Frameworks-and-Platforms]] | Agent SDKs, LangGraph, OpenAI Agents SDK, ADK, Pydantic AI, CrewAI, Managed Agents; decision guide |
| 07 | [[07-Agent-Reliability-Safety-and-Security]] | failure modes, human-in-the-loop, permissions, sandboxing, prompt injection, lethal trifecta, OWASP agentic Top 10 |
| 08 | [[08-Agent-Evaluation-and-Observability]] | trajectory evals, pass^k, benchmarks & their flaws, simulated users, tracing, production monitoring |
| 09 | [[09-Build-an-Agent-From-Scratch]] | minimal agent with tools, memory, budgets, approvals, an MCP server |
| 10 | [[10-Exercises]] | 80+ exercises |
| 11 | [[11-QA-Interview-Questions]] | 60 Q&A |
| 12 | [[12-Resources-and-Practice-Links]] | docs, papers, benchmarks, courses |
| 13 | [[13-Projects-and-Checkpoint]] | 6 projects + self-assessment |

## Weekly plan (5 weeks)
| Week | Focus | Output |
|---|---|---|
| 1 | Notes 01, 02, 09 | hand-written agent loop with 4 tools, budgets, approval gate |
| 2 | Note 03 | your own MCP server + client integration; compare to native tools |
| 3 | Notes 04, 05 | memory + compaction; orchestrator–worker research agent |
| 4 | Notes 07, 08 | threat model, sandbox, red-team; trajectory eval suite with pass^k |
| 5 | Note 06 + capstone | framework port (LangGraph or an SDK) and [[13-Projects-and-Checkpoint\|capstone agent with evals & guardrails]] |

## Completion checklist
- [ ] Explain agent vs workflow and pick the simplest architecture for a given task with justification
- [ ] Write a complete agent loop by hand: parallel tools, errors, stop conditions, step/cost/time budgets
- [ ] Design a tool set (names, descriptions, results) and evaluate tool-selection accuracy
- [ ] Build and test an MCP server (tools + resource), connect it to a client, and explain its auth model
- [ ] Implement memory (file/DB), context compaction, and checkpoint/resume
- [ ] Build an orchestrator–worker system and measure when multi-agent beats single-agent (and its token cost)
- [ ] Threat-model an agent using the lethal trifecta; implement least-privilege tools, approvals, sandboxing
- [ ] Red-team an agent against prompt injection and fix what breaks
- [ ] Build an agent eval: task set, graders, trajectory checks, pass^k over repeated trials
- [ ] Trace an agent run and diagnose a failure from the transcript
- [ ] Port your agent to one framework and explain what you gained/lost
- [ ] Capstone shipped on GitHub with eval CI and a security note

Back to [[../00-Roadmap]].
