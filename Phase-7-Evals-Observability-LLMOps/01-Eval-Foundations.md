---
tags: [phase-7, evals, foundations]
---
# 01 — Eval Foundations

> **An eval is a test suite for a probabilistic system.** Without it you can't tell whether a prompt tweak, model upgrade, or retrieval change helped — you only get vibes and anecdotes. With it you get a **flywheel**: observe failures → add test cases → improve → verify → ship → observe again. (Builds on [[../Phase-2-ML-Deep-Learning/04-Evaluation-and-Metrics]], [[../Phase-5-RAG-Knowledge-Systems/07-RAG-Evaluation-and-Production]], [[../Phase-6-Agents-Tools-MCP/08-Agent-Evaluation-and-Observability]].)

## 1. Why evals fail in practice
- Teams start with **generic metrics** (BLEU, "helpfulness 1–5") that don't reflect *their* failure modes.
- They build evals **before looking at data** (no error analysis), so the evals test imagined problems.
- They treat evals as a **one-time benchmark**, not a living artifact.
- They trust **uncalibrated LLM judges** and report **point estimates without uncertainty**.
- They evaluate the final answer only, missing wasteful/unsafe paths — or only the path, penalizing valid alternatives.
- There's no **owner**, no CI integration, no link to production.

## 2. Eval-driven development (EDD)
Like TDD for LLM apps:
1. **Define "good"** with the domain expert (product + SME).
2. **Write the first evals before/with the first prompt** (even 20 cases).
3. Build; run evals; **read failures**; refine prompts/tools/retrieval/model.
4. **Add every discovered failure** as a permanent test (regression suite).
5. **Gate releases** on evals in CI.
6. **Monitor production** with the same graders on sampled traffic ([[03-Observability-and-Monitoring]]).
Result: changes become *measured experiments* instead of arguments.

## 3. Start with error analysis (the highest-leverage activity)
1. **Collect ~100+ real traces** (production logs, beta users, SME-written scenarios; if none, generate realistic synthetic inputs across personas/scenarios/edge cases).
2. **Read them.** For each, note what went wrong in plain language (open coding) — "ignored return policy window," "hallucinated order ID," "tone too curt," "called wrong tool."
3. **Cluster into a taxonomy** of failure modes (axial coding): group notes, name categories, count frequencies.
4. **Prioritize** by frequency × severity × fixability.
5. **Turn each important category into an eval:** a code check if possible (regex/schema/state), otherwise a **narrow binary LLM-judge** with a written definition (one judge per failure mode, not one mega "quality" score).
6. Repeat periodically — new failure modes appear as the product and users evolve.
**Why binary and specific?** "Does the response cite a policy clause that actually exists? Pass/Fail" is checkable and actionable; "quality 1–5" isn't.

## 4. Anatomy of an eval
| Piece | Description |
|---|---|
| **Test case** | input (+ context/environment) + optional reference/expected behavior + metadata (slice tags, difficulty) |
| **System under test** | the full pipeline (prompt version, model, retrieval, tools) — evaluate *the system*, not the prompt in isolation |
| **Run** | execute cases (often N trials), capture outputs + traces + cost/latency |
| **Grader(s)** | code, LLM-judge, human, or hybrid → scores/labels + rationale |
| **Aggregation** | metrics with confidence intervals, by slice |
| **Report/decision** | compare to baseline; pass/fail thresholds; artifacts for debugging |

## 5. Test set construction
- **Sources:** real production queries (best), SME-authored scenarios, bug reports, adversarial/red-team cases, public benchmarks (for sanity only), **synthetic** generation (LLM-written variations over a *structured grid* of persona × task × difficulty × edge condition — then human-filter).
- **Coverage:** common paths, rare-but-costly cases, ambiguity (expected: clarify/abstain), out-of-scope (expected: refuse/redirect), safety/injection, multilingual, long inputs, malformed inputs, multi-turn flows.
- **Labels/expectations:** exact reference, key facts that must appear, forbidden content, required tool calls/final state, or rubric. Prefer **checkable expectations**.
- **Size:** start small (20–50) and grow; size needed depends on the effect you must detect (see §8).
- **Splits:** **dev** set for iteration vs **held-out test** set for release decisions (avoid overfitting prompts to the eval; refresh test periodically). Keep a **canary/"sentinel"** set of must-never-fail cases.
- **Versioning:** datasets are versioned artifacts (hash, changelog); never silently edit a set that has historical scores.
- **Privacy:** scrub PII from production-derived cases; access controls.
- **Contamination hygiene:** don't put test cases in prompts/few-shots/fine-tuning data.

## 6. Grader types (cheapest reliable first)
| Grader | Use for | Notes |
|---|---|---|
| **Exact/regex/schema** | format, IDs, enums, JSON validity | deterministic |
| **Programmatic state checks** | agent outcomes (DB row, file diff, API call log) | best for agents |
| **Reference-based fuzzy** | numeric tolerance, set overlap, F1, embedding similarity | similarity ≠ correctness; sanity-check |
| **Execution-based** | code (unit tests), SQL (result equality) | strong signal; sandbox |
| **LLM-as-judge** | tone, completeness, faithfulness, reasoning quality | calibrate! ([[02-LLM-as-Judge-and-Eval-Methods]]) |
| **Human review** | calibration, ambiguity, safety, launch gates | costly; sample smartly |
| **Pairwise preference** | comparing two versions | position-randomize |
| **User behavior metrics** | online: acceptance, edits, escalations, retention | noisy but real |
Combine: **code for hard constraints + judges for soft quality + humans for calibration.**

## 7. Metrics by task type
| Task | Primary metrics |
|---|---|
| Classification/routing | accuracy, per-class precision/recall, macro-F1, confusion matrix, calibration |
| Extraction | field-level precision/recall/F1, exact match, straight-through rate |
| Summarization | faithfulness (no unsupported claims), coverage of key points, length compliance |
| RAG QA | retrieval recall@k, correctness, faithfulness, citation validity, abstention accuracy |
| Chat/support | task resolution, policy compliance, tone, escalation correctness, safety |
| Code generation | tests pass (pass@k), lint/type checks, review quality |
| Text-to-SQL | execution accuracy, result-set equality, safety violations |
| Agents | outcome success, **pass^k**, steps/tokens/cost, constraint violations |
| Safety | attack success rate, refusal precision/recall, over-refusal rate |
| System | latency (TTFT/total p50/p95), cost per task, error rate |
Always report **cost and latency alongside quality** — the product decision is a trade-off frontier.

## 8. Statistics for evals (don't skip)
- **Variance sources:** model sampling, prompt sensitivity, dataset sampling, judge noise. Run **multiple trials** where outputs are stochastic.
- **Report uncertainty:** with `n` binary outcomes, `SE ≈ √(p(1−p)/n)`; 95% CI ≈ ±1.96·SE. At n=100, p=0.8 → ±~8 points; **a 3-point "improvement" is noise.** Use **bootstrap CIs** and report n.
- **Paired comparisons:** compare versions on the **same cases** (paired bootstrap / McNemar's test for binary; paired t/Wilcoxon for scores) — far more powerful than comparing unpaired means.
- **Effect size & sample size:** to detect a Δ=5pt improvement from p≈0.8 with paired data you typically need a few hundred cases; use power calculations or simulation.
- **Multiple comparisons:** many variants × many metrics → false discoveries; pre-register primary metric; confirm winners on the held-out test.
- **Slices:** report by segment; look for regressions hidden by averages (Simpson's paradox).
- **Judge uncertainty** adds error — propagate it ([[02-LLM-as-Judge-and-Eval-Methods]]).
- **Goodhart's law:** optimizing a metric degrades it as a proxy — rotate/refresh, keep held-out sets, include human spot-checks.

## 9. Offline vs online evaluation
| | Offline (pre-release) | Online (production) |
|---|---|---|
| Data | curated test sets | live traffic |
| Pros | fast, repeatable, safe, comparable | real distribution, real outcomes |
| Cons | may not match reality; goes stale | slower, noisy, risk exposure |
| Methods | CI evals, benchmark runs | sampled judge scoring, A/B tests, shadow, user signals |
**Both are required.** Offline gates what ships; online discovers what offline missed and refills the test set.

## 10. Evals in CI/CD
- **Tiers:** (1) *unit/deterministic checks* on every commit (parsers, tool schemas, guardrail regexes); (2) *smoke eval* (20–50 cases, minutes, cheap model/cached) on every PR; (3) *full eval* nightly/pre-release (hundreds–thousands of cases, multiple trials); (4) *post-deploy canary evals/monitors*.
- **Gate rules:** no statistically significant regression on primary metrics; **zero** failures on must-never-fail safety/policy cases; cost/latency budgets respected.
- **Flaky evals:** run multiple trials, seed where possible, cache deterministic parts (retrieval results), pin model versions/judge versions, use CIs not point thresholds, quarantine and fix unstable cases.
- **Cost control:** cheaper judge for smoke runs, caching, sampling, Batch APIs for nightly runs, prompt caching for shared prefixes.
- **Artifacts:** store full outputs/traces/graders' rationales for debugging; diff against baseline; link to prompt/model versions.

## 11. Benchmarks vs your evals
Public benchmarks (MMLU-like, SWE-bench-like) help **shortlist models**; they don't predict your task — they suffer **contamination, saturation, scaffold differences, and metric gaming**. Your task-specific, error-analysis-driven evals are the source of truth. Use frameworks (lm-evaluation-harness, HELM, Inspect) for standard suites and your own harness for product evals.

## 12. Organizational practices
- **Owner & cadence:** a named owner (often the AI engineer + a domain expert); weekly eval review.
- **SME involvement:** domain experts define good, label calibration sets, review failures — budget their time.
- **Transparency:** share dashboards and failure galleries with stakeholders; use eval results in launch decisions and customer reporting (FDE).
- **Eval debt:** untested features rot; track coverage of failure taxonomy by test cases.

## 13. Common mistakes checklist
Generic metrics only · no error analysis · single score for "quality" · uncalibrated judge · no CIs · tuning on the test set · not versioning datasets · ignoring cost/latency · no regression suite · evals never updated from production · eval leakage into prompts · treating public benchmark rank as product quality.

## Self-check
1. Walk through an error-analysis session and what it produces. 2. Why binary, narrow judges over 1–5 "quality"? 3. At n=150 and p=0.82, what's the 95% CI? Is +4 points significant? 4. How do you make evals non-flaky in CI? 5. Design a three-tier eval plan for a support copilot. 6. Why use paired comparisons?

Next → [[02-LLM-as-Judge-and-Eval-Methods]]
