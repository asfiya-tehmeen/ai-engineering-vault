---
tags: [phase-6, evals, observability, benchmarks]
---
# 08 — Agent Evaluation & Observability

> Agents are non-deterministic, multi-step, tool-using systems — evaluating them is harder than evaluating a prompt. Without evals you will **fly blind**, "fix" one behavior and break three, and can't tell whether a new model or framework is actually better. (General eval foundations: [[../Phase-2-ML-Deep-Learning/04-Evaluation-and-Metrics]], [[../Phase-5-RAG-Knowledge-Systems/07-RAG-Evaluation-and-Production]]; Phase 7 goes deeper.)

## 1. Vocabulary
**Task** (one test: input + environment + success criteria) · **trial** (one run of the agent on a task) · **transcript/trace** (full record of messages, tool calls, results, reasoning summaries) · **outcome** (the final state of the environment — what actually changed) · **grader** (code, model, or human scoring a trial) · **eval suite** (collection of tasks) · **harness** (infra running agent + graders) · **capability eval** (what can it do? hard tasks, low pass rate) vs **regression eval** (does it still do what it did? ~100% pass rate).

## 2. What to measure (levels)
| Level | Question | Metrics |
|---|---|---|
| **Component** | Is each tool/prompt good? | tool-selection accuracy, argument validity, retrieval recall, parse success |
| **Step/trajectory** | Is the path sensible? | redundant calls, wrong-order steps, loops, recovery from errors, adherence to policy, efficiency (steps/tokens) |
| **Outcome** | Did the environment end in the desired state? | task success (state check), partial credit, constraint violations |
| **Reliability** | Is it consistent? | **pass@k** (≥1 success in k), **pass^k** (all k succeed) |
| **System** | Cost & speed | tokens/$ per task, latency p50/p95, tool failures |
| **Safety** | Does it stay in bounds? | attack success rate, permission violations, unsafe actions, refusal correctness |
| **User-level** | Are people better off? | acceptance rate, edits, escalations, CSAT, time saved |
**Grade outcomes, not just paths** — agents may find valid unexpected routes; over-penalizing deviations from a "golden trajectory" is a common mistake. But **do** check hard constraints (no forbidden action, policy followed).

## 3. Reliability metrics: pass@k vs pass^k
- **pass@k:** probability at least one of k independent attempts succeeds (good for "can it ever solve this?", e.g., code with unlimited retries).
- **pass^k** (popularized by τ-bench): probability **all k** attempts succeed — measures **consistency**, which is what customer-facing deployments need. Example: pass@1 = 70% can coexist with pass^4 ≈ 30% — the same agent solves the same task reliably only 30% of the time. Report **pass^k** (k=3–8) for production-bound agents; use many tasks × multiple trials to estimate with CIs.
- Variance sources: sampling, tool nondeterminism, environment state, timing, user-simulator randomness → **reset environments** between trials; seed where possible; log versions.

## 4. Graders (use the cheapest reliable type)
1. **Code-based / state-based** (best): unit tests pass, DB row exists with right values, file diff matches, API call sequence satisfies constraints, schema valid, regex. Fast, objective, reproducible — design tasks so success is checkable.
2. **LLM-as-judge**: rubric-based scoring for qualities code can't check (helpfulness, tone, reasoning soundness, completeness). Use clear rubrics with anchors, require evidence quotes, pin the judge model/version, calibrate against human labels, beware biases (verbosity, position, self-preference) and **judge drift when models change** (research on version-dependent judge error). Prefer pairwise/rank for comparisons.
3. **Human review**: gold standard for calibration, ambiguous quality, safety; sample strategically (failures, disagreements, high-stakes).
4. **Hybrid:** code for hard constraints + judge for soft quality + human spot-checks.
Avoid **grader leakage** (agent can see/modify tests), **checkable-but-gameable** metrics, and graders that reward empty or degenerate outputs (benchmark audits have found tasks where empty responses pass, or simulated-user quirks can be exploited).

## 5. Building an agent eval suite (step by step)
1. **Start with real failures and real tasks** — 20–50 is enough to begin (don't wait for hundreds). Sources: support logs, manual tests, bug reports, user feedback, domain experts.
2. **Write unambiguous tasks** — two experts should agree on pass/fail; include required constraints; make impossible/ambiguous cases explicit (expected behavior: ask/abstain/escalate).
3. **Balance the set:** easy/hard, common/edge, should-act and should-refuse, adversarial/injection cases, different tools/paths.
4. **Build the environment:** sandbox with seeded data (mock DB, fake APIs, test repo), deterministic reset, realistic tool latency/failure injection.
5. **Simulated users** for conversational agents (LLM plays the user with a scenario/goal/persona; fixed seeds); validate the simulator.
6. **Run multiple trials per task;** store full transcripts and final states.
7. **Grade;** compute task success, pass^k, cost, latency, step counts; **slice** by task type.
8. **Read transcripts** — the highest-yield habit. Many "failures" are grader bugs or ambiguous tasks; many "successes" are lucky/unsafe.
9. **Fix → rerun → compare** with confidence intervals; keep a **regression set** (tasks that must always pass) separate from the **capability set** (tasks you're improving on). Watch for saturation: when pass rate hits ~100%, add harder tasks.
10. **Automate in CI** (smoke subset per PR, full nightly), track trends and cost.
11. **Feed production failures back** into the suite (flywheel).
Tools: **Inspect AI** (UK AISI), Braintrust, LangSmith, Langfuse, Phoenix, Promptfoo, OpenAI Evals, DeepEval, τ-bench harness, Terminal-Bench harness, custom pytest + sandbox.

## 6. Public agent benchmarks (know them — and their flaws)
| Benchmark | Measures | Notes (late-2026 sources — verify on official leaderboards) |
|---|---|---|
| **SWE-bench Verified** | resolve real GitHub issues in Python repos | widely reported through 2025; **2026 sources report contamination/flawed tests** (reports include a lab ceasing to report it; studies of solution leakage); treat as weak signal now |
| **SWE-bench Pro / newer variants**; **Terminal-Bench** | harder, multi-language/terminal-based software tasks | emerging replacements; check task provenance & contamination |
| **τ-bench / τ²-bench** | customer-service agents with tool APIs + simulated users + policies; **pass^k** | good template for conversational agent evals; audits have found exploitable scoring quirks |
| **OSWorld (Verified / 2.0)** | computer-use across real OS apps | earlier versions reported near/above human baselines; harder successors separate frontier agents |
| **WebArena / WebVoyager / BrowseComp** | browser tasks / hard web research | BrowseComp tests persistent search |
| **GAIA** | general assistant tasks needing tools | multi-step reasoning + tools |
| **AgentBench, ToolBench/BFCL (Berkeley Function-Calling Leaderboard)** | tool use & function calling | BFCL for call accuracy |
| **MLE-bench, PaperBench, RE-Bench, METR time-horizon evals** | long-horizon ML/research/engineering tasks; task "time horizon" trends | useful for capability trends |
| **AgentDojo** | prompt-injection robustness for tool agents | security eval |
**How to use benchmarks:** as *rough capability signals* and for model screening — never as proof for your use case. Pitfalls: **contamination**, **saturation**, **harness/scaffold differences** (scores depend heavily on the agent scaffold, tools, time/tokens allowed, retries), cherry-picked settings (pass@1 vs pass@k, subset of tasks), benchmark bugs, gaming. **Your own task suite on your environment is the real eval.**

## 7. Observability (tracing agents)
You can't debug what you can't see. Capture **per-run traces** with spans for: LLM calls (model, params, prompt version, input/output/cached/thinking tokens, latency, stop reason), tool calls (name, args, result size, latency, error), retrieval steps, sub-agent runs (parent/child), approvals, memory reads/writes, state checkpoints, costs.
- **Standards/tools:** OpenTelemetry (GenAI semantic conventions), Langfuse, LangSmith, Arize Phoenix, Braintrust, Logfire, Helicone, Datadog LLM Observability; provider consoles (session viewers with per-tool stats/cost) — use whatever your platform offers *and* export to your own backend for retention/governance.
- **Correlation IDs** across agents/tools/services; link traces to user feedback and eval runs.
- **Views you need:** timeline/waterfall of a run, token/cost breakdown by step, tool error rates, loop detection, "stuck" detection, per-tenant cost, success funnel by task type, latency percentiles.
- **Replay & debugging:** reproduce a run from the trace (checkpoint restore), modify a prompt/tool, rerun; diff two runs.
- **PII/secrets:** redact before logging; access control on traces; retention policy.

## 8. Production monitoring & feedback loops
- **Online checks on sampled traffic:** LLM-judge scoring against rubrics, policy-violation detectors, loop/budget alerts, tool-error spikes, escalation rate, "task abandoned" signals.
- **User feedback** (👍/👎, edits/undo, handoff requests) → triage queue → new eval cases.
- **Canary/shadow** new models/prompts/tools; compare to baseline on live-like traffic; automatic rollback triggers.
- **Drift:** upstream API changes, model updates, data shifts — contract tests and scheduled eval runs.
- **Cost & capacity:** budgets by tenant/feature; anomaly alerts; queue depth.
- **Incident response:** classify (security vs quality vs outage), contain (kill switch), root-cause from traces, add regression test, post-mortem.

## 9. Common eval mistakes
Waiting for a huge dataset before starting · only measuring final answers (missing unsafe or wasteful paths) · only measuring paths (penalizing valid alternatives) · single-trial results · no environment reset (state leakage between trials) · agent can read/modify the grader or tests · LLM-judge never calibrated · tasks ambiguous/impossible by accident · ignoring cost/latency · optimizing the benchmark not the product · not reading transcripts.

## 10. Hands-on mini-lab (recommended)
Build a 40-task eval for a support agent with mock tools (order lookup, refund, policy search): code graders on final DB state + policy-violation checks; simulated user; 5 trials/task; report pass@1, **pass^5**, cost/task; then change one thing (tool description, model, effort, memory) and show the delta with CIs.

## Self-check
1. Define pass@k vs pass^k and when each matters. 2. Why grade outcomes over trajectories — and what must you still check in the trajectory? 3. How do you design an eval environment that resets between trials? 4. Name three flaws of public agent benchmarks. 5. What goes into an agent trace? 6. How do production failures flow back into evals?

Next → [[09-Build-an-Agent-From-Scratch]]
