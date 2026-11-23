---
tags: [phase-6, exercises]
---
# 10 — Exercises (Phase 6)

🟢 easy · 🟡 medium · 🔴 hard. **Cost & safety rules:** every agent run must have **step, token/$, and wall-clock budgets**; run code only in a **container/sandbox with no real credentials**; use small models for iteration; never point an experimental agent at production systems. Keep code in `phase6-exercises/` and save **transcripts** of every run (you'll learn most from reading them).

## A. Agent fundamentals (Note 01)
- [ ] 🟢 A1. For 8 tasks (classify tickets, book a trip, fix a failing test, summarize a PDF, process refunds, research a company, fill a web form, nightly report) decide: single call / workflow / agent. Justify with the four checks (complexity, value, viability, cost of error).
- [ ] 🟢 A2. Implement each of the five workflow patterns on a toy task (chain, route, parallel, orchestrator–workers, evaluator–optimizer); compare quality, cost, latency.
- [ ] 🟡 A3. Write a minimal agent loop (no framework) with `max_steps`, token and time budgets, repeated-call detection; show it failing gracefully when budgets hit.
- [ ] 🟡 A4. Error compounding experiment: a 10-step synthetic task where each step has a controllable failure probability; measure end-to-end success vs per-step accuracy; then add verification/retry and measure the improvement.
- [ ] 🟡 A5. Plan-and-execute vs pure ReAct-style on 20 multi-step tasks (e.g., data wrangling); compare success, steps, tokens, and recovery from surprises.
- [ ] 🔴 A6. Build a **verification-driven coding agent** for small bug-fix tasks (given a repo with failing tests): tools = read/edit/run_tests; measure pass rate and steps; ablate "run tests after every edit".
- [ ] 🔴 A7. Add reflection: after finishing, the agent self-reviews against a checklist; measure whether it catches real mistakes vs adds noise.

## B. Tool design & environments (Note 02)
- [ ] 🟢 B1. Take a REST API with 12 endpoints; design 4 goal-oriented tools instead of 12 thin ones. Write descriptions; justify consolidation.
- [ ] 🟡 B2. **Tool description A/B:** vague vs precise descriptions for 6 tools; 50 test queries; measure tool-selection accuracy and argument validity.
- [ ] 🟡 B3. Implement `response_format: concise|detailed` and pagination on a verbose search tool; measure tokens and task success.
- [ ] 🟡 B4. Safe file tools: confine paths to a root; write tests for `../`, symlinks, absolute paths, Unicode tricks.
- [ ] 🟡 B5. Promote a risky bash action to a dedicated tool with a gate and typed args; show what the harness can now do (approve, audit, parallelize).
- [ ] 🟡 B6. Implement **tool search**: 200 mock tools; BM25/embedding search over descriptions; agent loads only matched schemas; compare tokens/accuracy vs listing all tools.
- [ ] 🔴 B7. **Programmatic tool calling** experiment: a task needing 30 tool calls with large intermediate data; compare standard loop vs code-sandbox composition on tokens, latency, accuracy.
- [ ] 🔴 B8. Build a **sandboxed code-execution tool** (Docker: no network, read-only FS except /work, CPU/mem/time limits, non-root); try escape attempts; document residual risks.
- [ ] 🔴 B9. **Browser agent** (Playwright): DOM-based tools (`goto`, `click_selector`, `get_text`) vs screenshot-based loop on 15 web tasks (local test site you control); compare reliability, cost, speed.
- [ ] 🔴 B10. Write a **Skill** (folder with `SKILL.md` + script) for a repeatable procedure; show progressive disclosure keeps base context small; evaluate with/without the skill.
- [ ] 🔴 B11. Build a tool-improvement loop: collect failed transcripts, have a model propose better descriptions, test on held-out tasks; report overfitting risk and gains.

## C. MCP (Note 03)
- [ ] 🟢 C1. Build an MCP server (stdio) with 3 tools, 1 resource, 1 prompt; test with the MCP Inspector.
- [ ] 🟡 C2. Connect your server to a desktop/IDE client and to your own agent via the Python SDK; verify tool calls and results.
- [ ] 🟡 C3. Add input validation, path confinement, parameterized SQL, timeouts, and output caps; write unit tests including malicious inputs.
- [ ] 🟡 C4. Compare **MCP vs native function tools vs CLI+bash** for the same 10 tasks (tokens, latency, accuracy, security gating ability).
- [ ] 🟡 C5. Implement tool annotations (`readOnlyHint` etc.); explain why hints aren't security controls; add real enforcement server-side.
- [ ] 🔴 C6. Expose your server over **Streamable HTTP** with OAuth-style bearer auth (or a gateway); enforce **per-user authorization** against a mock backend; test a confused-deputy attempt.
- [ ] 🔴 C7. **Tool-poisoning lab:** craft a malicious tool description that tries to hijack the agent; demonstrate; implement defenses (description review/pinning/diff alerts, allow-lists, approvals).
- [ ] 🔴 C8. **Rug-pull test:** server changes a tool's behavior after approval; build a client-side hash/pin check that detects and blocks it.
- [ ] 🔴 C9. Build an **MCP gateway** (proxy) that adds auth, per-tool policy, rate limits, audit logs, and schema pinning in front of 2 servers.
- [ ] 🔴 C10. Implement **sampling or elicitation** in a server (e.g., ask the user to confirm a destructive action via elicitation); handle clients lacking the capability.
- [ ] 🔴 C11. Read the current MCP spec's authorization section; write a 1-page summary of the flow (discovery, PKCE, resource indicators, token audience) and a sequence diagram; list what changed in the latest revision.

## D. Memory, context & state (Note 04)
- [ ] 🟢 D1. Add a todo/plan file tool; measure effect on long-task completion (A/B on 15 tasks).
- [ ] 🟡 D2. File-based memory (`memories/` + index): agent reads at start, writes learnings; test recall of a preference after 40 unrelated turns.
- [ ] 🟡 D3. Context experiments: (a) no management, (b) clear stale tool results, (c) compaction; on a 100-step task measure tokens, cost, accuracy, and **fact survival** with probe questions.
- [ ] 🟡 D4. Offload large tool outputs to files; agent reads slices; compare against inline results.
- [ ] 🟡 D5. Checkpointing: persist state after every step; kill the process randomly; resume correctly with **idempotent tools** (prove no duplicate side effects).
- [ ] 🔴 D6. **Long-running harness:** initializer + incremental sessions building a small app from a feature list; each session verifies, updates the list/progress log, commits; measure completion across fresh contexts and failure modes (premature "done," broken states).
- [ ] 🔴 D7. **Memory poisoning drill:** hostile document persuades the agent to store a malicious "preference"; show persistence; implement mitigations (provenance, untrusted-source write confirmation, sanitization); re-test.
- [ ] 🔴 D8. Run an agent on a **durable execution engine** (Temporal/Inngest/Restate or a Postgres-backed queue) with a human-approval wait of hours; crash workers; verify resume.
- [ ] 🔴 D9. Implement hierarchical memory (core block always in context + archival search tool); evaluate recall vs context cost.

## E. Multi-agent (Note 05)
- [ ] 🟡 E1. Build an **orchestrator–worker research agent** (lead + 3 parallel workers with search/fetch tools); require structured, sourced outputs from workers.
- [ ] 🟡 E2. Single vs multi-agent experiment on 30 research questions (A: single; B: 3 workers; C: 8 workers): rubric quality, coverage, citation accuracy, tokens, cost, wall-clock. Plot quality vs cost.
- [ ] 🟡 E3. Vague vs detailed delegation prompts for workers; measure duplicate work and output quality.
- [ ] 🟡 E4. Handoff-based support system (triage → billing/tech/returns specialists); test context transfer strategies (summary vs full history).
- [ ] 🔴 E5. Reader/actor separation: quarantined reader (no tools) processes untrusted web content → schema-validated facts → privileged actor; red-team with injections; compare attack success vs a single-agent baseline.
- [ ] 🔴 E6. Evaluator–optimizer for code: generator + test-running critic loop; cap iterations; examine test-gaming attempts and add protections.
- [ ] 🔴 E7. Failure-mode catalog: induce 5 multi-agent failures (duplication, telephone game, runaway spawning, ping-pong, bad claim accepted); implement mitigations; show before/after metrics.
- [ ] 🔴 E8. Advisor pattern: cheap executor consults a stronger model at decision points; compare cost/quality vs always-strong and always-cheap.

## F. Frameworks & platforms (Note 06)
- [ ] 🟡 F1. Port your raw-loop agent to **LangGraph** (state, nodes, conditional edges, checkpointer, interrupt before write tools); compare LOC, debuggability, resume, HITL.
- [ ] 🟡 F2. Port to a **vendor SDK** (OpenAI Agents SDK or Claude Agent SDK) using its tools/handoffs/guardrails/hooks; compare.
- [ ] 🟡 F3. Port to **Pydantic AI** or **Google ADK**; note type-safety, tracing, and eval features.
- [ ] 🔴 F4. If you have access, deploy the agent on a **hosted agent runtime** (e.g., Managed Agents): versioned agent config, environment, session streaming, vault credentials, memory store, session budget; document setup vs self-hosting trade-offs and billing.
- [ ] 🔴 F5. Add **OpenTelemetry/Langfuse tracing** to all variants; compare trace quality.
- [ ] 🔴 F6. Write a **framework-portability layer**: prompts/tools/evals in framework-independent files; swap runtimes without changing tests; prove parity on the golden task suite.
- [ ] 🔴 F7. Prototype A2A or an agent-to-agent handoff between two independently deployed agents; list gaps vs MCP.

## G. Reliability, safety & security (Note 07)
- [ ] 🟢 G1. Threat-model 5 agents (email assistant, coding agent, support refunder, web researcher, DB analyst) with the **lethal trifecta**: which legs present, which to remove?
- [ ] 🟡 G2. Build an **injection test suite** (≥50 cases across web pages, emails, PDFs, tool outputs, images' alt-text, multi-turn); compute attack success rate on your agent; iterate defenses; report ASR vs utility retention.
- [ ] 🟡 G3. Implement **approval gates** with per-tool/arg policies (allow/ask/deny); measure approval fatigue by simulating 100 actions; batch low-risk.
- [ ] 🟡 G4. **Exfiltration via rendering:** make the agent emit a markdown image/link carrying data; implement output sanitization and egress allow-list; re-test.
- [ ] 🟡 G5. Budgets & kill switch: enforce step/$/time limits and an external stop signal; test a runaway loop and "denial of wallet" prompt.
- [ ] 🔴 G6. **Confused-deputy test:** agent with a service credential serving multiple users; prove user A cannot access user B's data via the agent; fix with delegated user-scoped tokens and backend authz.
- [ ] 🔴 G7. **Secrets isolation:** run a tool needing an API key without placing the secret in context or the sandbox (egress-injection proxy or scoped token broker); attempt to extract it via prompt injection.
- [ ] 🔴 G8. **Plan-then-execute / CaMeL-style control-flow integrity (lite):** privileged agent fixes a plan before reading untrusted data; deviations require approval; evaluate against injection suite.
- [ ] 🔴 G9. **Test tampering:** ensure the agent cannot edit its grader/tests (read-only mounts, hidden tests); show an attempted cheat being blocked.
- [ ] 🔴 G10. Write a **security design review** (3 pages) for a customer-facing agent: assets, threats (OWASP agentic Top 10 mapped), controls, residual risk, monitoring, incident response.
- [ ] 🔴 G11. Red-team an MCP server ecosystem scenario: two servers (private-data + outbound-capable) → exfil attempt via cross-server prompt injection; design a policy that prevents it.

## H. Evaluation & observability (Note 08)
- [ ] 🟢 H1. Write 25 tasks for a support-agent environment (mock order DB, refund API, policy docs); define pass/fail on **final state** plus policy constraints.
- [ ] 🟡 H2. Build the eval harness: resets environment, runs N trials per task, stores transcripts and final states, computes pass@1, pass@k, **pass^k**, cost/task with bootstrap CIs.
- [ ] 🟡 H3. Add a **simulated user** (LLM with scenario/persona); validate it behaves consistently; measure variance across seeds.
- [ ] 🟡 H4. Calibrate an **LLM judge** for soft criteria against 40 human labels; check agreement; test verbosity/position biases.
- [ ] 🟡 H5. **Transcript reading session:** read 20 failed trials; categorize (agent bug, tool bug, ambiguous task, grader bug); fix the top category; re-run.
- [ ] 🟡 H6. Instrument tracing; build a dashboard (success by task type, tool error rate, tokens per step, loops detected).
- [ ] 🔴 H7. Compare two models (or effort levels) on your suite: report quality, **pass^5**, cost, latency with CIs; make a deployment recommendation.
- [ ] 🔴 H8. **Benchmark audit:** pick a public agent benchmark; read 20 tasks and graders; list potential flaws (leakage, gameable checks, ambiguity); run a baseline and compare with published numbers; write findings.
- [ ] 🔴 H9. Build a **CI gate**: smoke eval subset on every PR; nightly full eval; fail on regression beyond CI; archive transcripts.
- [ ] 🔴 H10. Production flywheel: sample live traces → LLM-judge flags → human review → add to regression suite; document the loop and measure time-to-fix.

## I. Mini-capstones (pick 2)
- [ ] 🔴 I1. **Coding agent** that fixes issues in a sample repo with tests, sandboxed, with budgets, trajectory logging, and pass^k evals.
- [ ] 🔴 I2. **Support agent** (τ-bench-style) with policy compliance checks, simulated users, human handoff, and eval CI.
- [ ] 🔴 I3. **Research agent** (orchestrator–workers) with citations, verification pass, and cost/quality curve.
- [ ] 🔴 I4. **Data-analyst agent** (read-only SQL + code sandbox) with validation, chart outputs, and injection/exfil defenses.

## Cadence
3 exercises/week; a weekly "transcript review" hour; keep an `agent-lab-notebook.md` in the vault (hypothesis → change → result → decision). Review [[11-QA-Interview-Questions]] each Sunday.
