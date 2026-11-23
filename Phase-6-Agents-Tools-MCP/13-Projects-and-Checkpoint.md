---
tags: [phase-6, projects, checkpoint]
---
# 13 — Projects & Phase Checkpoint

> Each repo: README with **Problem · Autonomy level & why · Architecture diagram (agent, tools, sandbox, memory, approvals) · Threat model (trifecta analysis) · Eval suite & results (success, pass^k, cost, latency, safety ASR, with CIs) · Failure analysis from transcripts · How to run safely**. Include saved **sample transcripts** (sanitized). Hiring managers learn more from your transcript analysis than from your demo video.

## Project 1 — `agentcore`: framework-free agent runtime *(Notes 01, 02, 09)*
Tool registry with schemas and risk tiers, validated/sandboxed execution, parallel read-only calls, step/$/time budgets, repeat-call detector, approval gates, JSONL traces, checkpoint/resume, prompt caching, stale-result clearing. Ships with 5 sample tools and a CLI.
**Stretch:** pluggable model backends; per-tool timeout/retry policies; replay mode that re-executes a trace deterministically against mocks.

## Project 2 — `mcp-suite`: servers, gateway & client *(Note 03)*
Two MCP servers (e.g., `tickets`, `warehouse-sql`) with goal-oriented tools, validation, per-user authorization, annotations, structured outputs, resources, prompts; a **gateway** adding auth, per-tool policy, rate limits, audit, and **schema pinning** (detects rug pulls); a client integration into Project 1.
**Deliverables:** tool-selection eval (MCP vs native vs CLI), injection/tool-poisoning red-team results, a 1-page auth flow diagram aligned to the current spec.

## Project 3 — `research-swarm`: orchestrator–worker research agent *(Notes 04, 05)*
Lead plans and writes the plan to memory; 3–8 parallel workers (search/fetch tools) return structured, sourced findings via a shared store; lead synthesizes; verifier checks citations. Budgets and effort-scaling rules.
**Evaluation:** 30 research questions; single-agent vs multi-agent quality/cost/latency curve; citation accuracy; failure catalog (duplication, telephone game, runaway spawn) with mitigations. Report whether multi-agent *earned* its cost.

## Project 4 — `sandboxed-coder`: verification-driven coding agent *(Notes 02, 04, 07)*
Agent fixes failing tests in small repos inside a locked-down container (no network, no secrets, resource limits). Tools: read/edit/search/run_tests; **tests are read-only to the agent** (hidden tests for grading); git commits as undo/memory; long-task harness (task list + progress log).
**Evaluation:** 40 bug-fix tasks, pass@1 and **pass^3**, steps, cost; analysis of test-gaming attempts and protections; transcript review report.

## Project 5 — `support-agent-evals`: τ-bench-style evaluation harness *(Note 08)*
Mock retail/airline environment (DB + policy doc + tools), LLM **simulated users** with scenarios, code graders on final DB state + policy-violation checks, multi-trial runner with **pass^k**, bootstrap CIs, cost/latency, transcript viewer, CI gate. Compare two models or two prompt/tool versions and write a go/no-go memo.
**Stretch:** benchmark audit — find and document grader loopholes (e.g., empty-response passes), then fix the grader.

## Project 6 (Capstone) — `ops-copilot`: secure, evaluated enterprise agent *(everything)*
Choose a scenario: IT helpdesk automation, finance reconciliation assistant, customer-support refunder, or data-analyst agent.
**System requirements**
- Tools via your own **MCP servers** behind the gateway; read-only by default; writes gated by approvals with exact-action display.
- **Per-user delegated authorization** enforced in backends (confused-deputy test passes).
- **Trifecta analysis:** document which legs exist; remove or isolate (reader/actor separation, egress allow-list, sanitized rendering).
- Sandboxed execution; secrets never in context; budgets; kill switch; audit log.
- Memory with provenance and poisoning defenses; checkpoint/resume; durable human-approval waits.
- Observability: traces, dashboards (success funnel, tool errors, cost per task, loops).
- **Eval suite:** ≥60 tasks incl. injection and permission cases; pass^k; CI gate; nightly run.
**Customer-facing deliverables:** (a) scoping & autonomy-ladder plan (suggest-only → approve-to-act → bounded autonomy), (b) architecture & security overview with threat model and red-team results, (c) evaluation report with limitations, (d) operations runbook (incidents, rollback, adding tools, rotating credentials), (e) exec summary with ROI assumptions, (f) 5-minute demo video including a *failure* and how the system contains it.
This is a flagship FDE portfolio piece.

## Phase 6 Self-assessment (0–3; need ≥2 before Phase 7)
| Skill | 0–3 |
|---|---|
| Choose single call / workflow / agent with justification (four checks) | |
| Implement the five workflow patterns | |
| Write an agent loop with budgets, loop detection, approvals, checkpoints | |
| Design tools (descriptions, results, errors, consolidation) and evaluate selection accuracy | |
| Explain programmatic tool calling, tool search, skills | |
| Build/test an MCP server and explain host/client/server, primitives, transports | |
| Explain MCP authorization and MCP-specific threats (poisoning, rug pull, passthrough) | |
| Implement compaction/context editing/offloading and test fact survival | |
| Implement file/DB memory with write policy and poisoning defenses | |
| Make an agent crash-safe with idempotent tools and durable execution | |
| Build an orchestrator–worker system and measure multi-agent ROI | |
| Compare ≥2 frameworks/platforms and justify a choice | |
| Threat-model with the lethal trifecta; apply least agency | |
| Implement sandboxing, egress control, credential isolation | |
| Red-team for injection/exfil and report attack success rates | |
| Build an agent eval with outcome graders, simulated users, multi-trial pass^k | |
| Calibrate an LLM judge; read transcripts to find root causes | |
| Instrument tracing and build dashboards | |
| Critique public benchmarks (contamination, scaffold dependence) | |
| Plan a graduated-autonomy rollout for a customer | |

## Mock interview (60 min)
1. 10 min — "Design an agent to process insurance claims. Workflow or agent? Where does autonomy start and stop?"
2. 15 min — live: implement a tool loop with budgets and a repeated-call guard; discuss failure handling.
3. 15 min — security: "Our email assistant can read mail, access files, and send messages. What's the risk and the fix?" (trifecta, reader/actor, approvals, egress, per-user auth)
4. 10 min — eval: "How do you know v2 is better than v1?" (task suite, graders, pass^k, CIs, transcript review, cost)
5. 10 min — scenario: "Customer's agent worked in the pilot but failed 30% in production." (traces → categories → evals → fixes → canary)

## What you've unlocked
You can design, build, secure, and *measure* tool-using agents — and explain trade-offs to technical and executive audiences. Phase 7 goes deep on the operational discipline around all of it: **evals at scale, observability, LLMOps, fine-tuning/serving, security hardening, cost and quality management**.

⏸️ **Pause point.** Tell me when you're ready for **Phase 7 — Evals, Observability & LLMOps** (I'll research current eval tooling, LLM-as-judge practice, tracing standards, fine-tuning (LoRA/RL) and serving stacks, guardrails, and AI governance first).

Back to [[00-Phase-6-Index]] · [[../00-Roadmap]]
