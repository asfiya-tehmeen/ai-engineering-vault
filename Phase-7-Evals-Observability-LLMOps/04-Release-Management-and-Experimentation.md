---
tags: [phase-7, release, cicd, experimentation, llmops]
---
# 04 — Release Management & Experimentation (LLMOps)

> In LLM systems, **the prompt, model, tools, retrieval index, guardrails, and config are all "code"** — any change can alter behavior. LLMOps = applying software-delivery discipline (versioning, testing, staged rollout, rollback) to these artifacts, plus handling the unique problem that **model providers change underneath you**.

## 1. What to version (everything that affects behavior)
| Artifact | Practice |
|---|---|
| **Prompts/templates** | files in git (`prompts/support_v12.md`) or a prompt registry; semantic IDs; changelog; link to eval results |
| **Model & params** | explicit model IDs (no "latest" aliases in prod unless intentional); effort/thinking/max_tokens config; fallbacks |
| **Tool schemas & descriptions** | versioned with consumers; contract tests |
| **Retrieval/index** | embedder + chunker + index version; blue/green indexes |
| **Guardrail policies** | versioned rules/classifier versions |
| **Datasets** | eval/golden sets hashed and versioned |
| **Judge prompts/models** | pinned (judge drift!) |
| **Feature flags/config** | typed config with environments and audit trail |
| **Dependencies/SDKs** | lockfiles; pin client library versions |
Every production response/trace records the **version tuple** that produced it → reproducibility and bisecting regressions.

## 2. Prompts as code (practical patterns)
- **In-repo prompts** (Jinja2/Markdown templates + typed variables) reviewed via PRs → best for engineering-led teams.
- **Prompt registries/CMS** (Langfuse, LangSmith, Braintrust, PromptLayer, Humanloop-style) → non-engineers can iterate; require **environments (dev/staging/prod), versions, approvals, rollback**, and eval gates before promotion.
- Keep **business logic out of prompts** where possible (code does validation/routing); keep prompts small and focused; document intent and known failure modes at the top.
- **Prompt unit tests:** variable rendering, token counts within budget, snapshot tests for structure (not exact text), schema compliance on mock outputs.
- **Config-driven model selection:** `{route: "support_reply", model: "...", effort: "...", fallback: "..."}` in typed config so swaps don't need code changes.

## 3. CI/CD pipeline for an LLM app
```
PR opened → lint/type/unit tests → prompt/tool contract tests → smoke eval (cheap, 30–100 cases) → report on PR
merge → build artifact → deploy to staging → full eval (nightly or pre-release) + red-team suite + load test
→ approval (quality + safety + cost gates) → canary (1–5%) with online monitors → ramp → full rollout
post-deploy → scheduled eval + drift/regression monitors + golden probes
```
**Gates:** primary quality metrics non-inferior (CI-based), **zero** must-never-fail failures, safety ASR ≤ threshold, latency/cost budgets, schema validity ≥ threshold. Provide **override with justification** for urgent fixes (and follow-up).
**Artifacts:** eval reports, diffs vs baseline, trace samples, cost estimates, release notes.

## 4. Rollout strategies
| Strategy | How | Use |
|---|---|---|
| **Shadow (dark launch)** | run new version in parallel on live traffic; users see old output; compare offline/with judges | safest first step for model/prompt/retrieval changes |
| **Canary** | route small % of users/requests to new version; monitor; ramp | detect issues under real traffic |
| **A/B test** | randomized exposure with success metrics & statistics | measure business impact (CSAT, resolution, conversion) |
| **Blue/green** | two full environments/indexes; switch traffic atomically | index re-builds, infra changes |
| **Feature flags** | per-tenant/user toggles; kill switches | gradual enablement; instant rollback |
| **Interleaving** (ranking systems) | mix results from A and B in one list | search/ranking experiments |
| **Rainbow deployments** | many versions concurrently (stateful agents) | long-running agents can finish on their version |
**Rollback plan** must be tested: config flip to previous prompt/model, previous index, fallback path, or human handoff.

## 5. Online experimentation (A/B) basics
- **Unit of randomization:** user/session (avoid cross-contamination); **sample ratio mismatch (SRM) checks**; **pre-register** primary metric, guardrail metrics (safety, latency, cost), and stopping rule.
- **Power/duration:** estimate needed sample from baseline rate & minimum detectable effect; account for novelty/learning effects and weekly seasonality.
- **Metrics:** primary business metric (resolution rate, CSAT, task completion) + quality judges + cost/latency guardrails + safety counters.
- **Analysis:** CIs, sequential testing if peeking, segment analysis, **don't over-slice**; for LLM outputs, supplement with human review of a sample.
- **Interference:** shared caches, shared agents' memory, or network effects can leak between arms — design isolation.

## 6. Model migration & deprecation playbook
Providers release new models and **retire old ones on schedules**; behavior can change even within "same" names. Treat migrations as routine:
1. **Track deprecation notices** (subscribe; calendar the end-of-life dates; inventory which routes use which model IDs).
2. **Define the migration eval:** your regression + capability + safety suites; judge pinned; also cost/latency.
3. **Baseline** the current model; run the candidate on the same suite with N trials.
4. **Adapt prompts** — don't just swap the ID: newer models may need *less* scaffolding, have different parameter support (e.g., sampling params removed, thinking/effort settings, forced tool-choice limits, prefill removal), different token counts/tokenizers (re-estimate budgets), different refusal behavior. Prune "prompt cruft" written for older models and re-test.
5. **Shadow → canary → ramp** with monitors; compare cost per task.
6. **Document differences** and update fallbacks/routing.
7. **Keep rollback** until confidence; remove deprecated paths promptly.
Use **provider-specific migration guides** and capability APIs; re-baseline token counts.

## 7. Managing multiple models (routing & fallbacks in ops)
- **Routing config** (cheap→strong cascades, per-tenant premium tiers, language-specific models) with eval-backed thresholds.
- **Fallback chains** for outages/rate limits/refusals; log which model actually served (`served_by`).
- **Per-model prompt variants** where needed; avoid hidden coupling.
- **Capacity & quotas:** multi-key/provider pools, priority tiers/committed throughput for critical routes; rate-limit shaping; queue for non-interactive.

## 8. Data & experiment management
- **Experiment tracking:** record prompt/model/config → eval metrics → cost/latency → decision (MLflow, W&B, Langfuse experiments, Braintrust experiments, or a Git-tracked results table).
- **Dataset management:** versions, splits, provenance, PII status, owners; changelogs when cases are added/removed/relabeled.
- **Reproducibility:** pin versions; store raw outputs; record seeds where possible; accept residual nondeterminism (use multiple trials).

## 9. The data flywheel in release terms
Production failures (flagged by monitors/feedback) → triaged weekly → each becomes a **regression test** → fix shipped via the pipeline → test stays forever. Track: **time-to-detect, time-to-fix, regression escape rate** (bugs that reappear), and **coverage** of failure taxonomy by tests.

## 10. Organizational practices
- **RACI:** who owns prompts, evals, guardrails, on-call, vendor relationships.
- **Change management:** release notes, stakeholder demos with metrics, customer comms for behavior changes (FDE).
- **Freeze windows** for critical business periods; emergency change procedure.
- **Blameless post-mortems**; shared failure gallery.
- **Documentation:** runbooks, architecture, "how to add an eval," "how to roll back."

## 11. Environments & promotion
`dev` (cheap models, mocks) → `staging` (prod-like data/models, full evals) → `prod`. Separate keys/budgets per environment; prevent test traffic from polluting production metrics; use synthetic tenants for end-to-end tests.

## 12. Anti-patterns
Editing prompts directly in prod UI without eval/rollback · "latest" model aliases in production · no record of which version served a response · swapping models without re-tuning prompts · shipping on 5 anecdotes · A/B tests without guardrail metrics · eval-only gating with no online monitoring · no deprecation tracking · changing judge and system simultaneously (can't attribute) · unlabeled flags that live forever.

## Self-check
1. List 8 artifacts you'd version and how. 2. Design a CI/CD pipeline with quality, safety, and cost gates. 3. Shadow vs canary vs A/B — when each? 4. Write a model-migration plan for a provider deprecating your model in 90 days. 5. What pitfalls arise in A/B tests of LLM features? 6. How do you ensure rollback works?

Next → [[05-Fine-Tuning-and-Adaptation]]
