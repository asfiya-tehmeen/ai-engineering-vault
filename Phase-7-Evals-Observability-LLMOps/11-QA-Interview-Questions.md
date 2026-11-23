---
tags: [phase-7, qa, interview]
---
# 11 — Q&A (Phase 7)

> Answer aloud first, then expand.

## Evals
> [!question]- Q1. Why do LLM apps need evals?
> Outputs are probabilistic and changes (prompt, model, retrieval, tools) have non-obvious effects. Evals turn changes into measured experiments, catch regressions, gate releases, and feed a flywheel from production failures back into tests.

> [!question]- Q2. Where should you start building evals?
> Error analysis: read ~100 real traces, write failure notes, cluster them into a taxonomy, prioritize by frequency × severity, and turn important categories into targeted checks (code graders or narrow binary judges).

> [!question]- Q3. Why prefer narrow binary judges to a "quality 1–5" score?
> They're more reliable, actionable, and calibratable against human labels; 1–5 scores cluster mid-scale, mix criteria, and are hard to act on.

> [!question]- Q4. What's eval-driven development?
> Define "good" with SMEs, write first evals alongside the first prompt, iterate by reading failures, add every discovered failure as a regression test, gate releases on evals in CI, and monitor production with the same graders.

> [!question]- Q5. How do you build a good test set?
> Real production queries (best) plus SME scenarios, adversarial and out-of-scope cases, and structured synthetic variations (human-filtered); label with checkable expectations; version it; split dev vs held-out test; keep sentinel must-never-fail cases; scrub PII.

> [!question]- Q6. How big must an eval set be?
> Depends on the effect to detect: at n=100 and p≈0.8 the 95% CI is about ±8 points, so small gains are noise. Use paired comparisons and power calculations; grow the set as the product matures.

> [!question]- Q7. Why paired comparisons?
> Both versions run on the same cases, so case-difficulty variance cancels; paired bootstrap/McNemar tests are much more powerful than comparing unpaired means.

> [!question]- Q8. How do you handle flaky evals in CI?
> Multiple trials, seeded/cached deterministic parts, pinned model/judge versions, CI-based thresholds instead of point cutoffs, quarantine unstable cases, and tiered suites (cheap smoke on PR, full nightly).

> [!question]- Q9. Offline vs online evaluation?
> Offline (curated sets) gates releases and is repeatable; online (sampled judging, A/B tests, user signals) reflects real distribution and discovers new failures. Both are needed and should share graders.

> [!question]- Q10. Why are public benchmarks insufficient for product decisions?
> Contamination, saturation, scaffold/harness differences, metric gaming, and mismatch with your task/data. Use them to shortlist models; use your own evals to decide.

> [!question]- Q11. What is Goodhart's law in this context?
> When a measure becomes a target it stops being a good measure — optimizing prompts against a small fixed eval overfits; mitigate with held-out sets, refreshed cases, multiple metrics, and human spot-checks.

## LLM-as-judge
> [!question]- Q12. How do you calibrate an LLM judge?
> Label a calibration set with experts (≥~50, ideally 100+, stratified), split dev/test, run the judge, compute confusion matrix, TPR/TNR (or precision/recall for the fail class), agreement and κ, iterate the prompt on dev errors, confirm on test, and recalibrate when the judge/prompt/task changes.

> [!question]- Q13. Why isn't accuracy enough for a judge?
> With imbalanced labels a judge that passes everything looks accurate. Report per-class metrics (TPR/TNR) and agreement beyond chance (κ).

> [!question]- Q14. How do you correct an aggregate pass rate for judge error?
> With measured TPR and TNR: θ̂ = (p_obs + TNR − 1)/(TPR + TNR − 1) (Rogan–Gladen); bootstrap both eval items and calibration pairs for CIs; avoid absolute claims if TPR+TNR is near 1.

> [!question]- Q15. Name four judge biases and mitigations.
> Position (randomize/both orders), verbosity (rubric/length control), self-preference (cross-family judge), leniency (check TNR with known-bad examples), prompt sensitivity (variants/ensembles), drift (pin versions, frozen calibration set).

> [!question]- Q16. Pairwise vs absolute judging?
> Pairwise is more sensitive for comparing versions but needs order randomization and says nothing about absolute quality; absolute checks catch "both bad." Use both.

> [!question]- Q17. How do you control judge cost?
> Sample (e.g., 5–20% of traces plus errors/outliers), use smaller judges with cascades to a strong adjudicator, cache, batch, shorter rubrics, and score only the relevant slices.

> [!question]- Q18. Risks of synthetic eval data?
> Mirrors generator style, skews easy, leaks source wording (inflating retrieval metrics), misses real user quirks. Mix with real traffic cases and measure the synthetic–real gap.

## Observability
> [!question]- Q19. What do you log for every LLM call?
> IDs (trace/session/tenant), version tuple (prompt/model/tools/index/guardrails), inputs/outputs (redacted), retrieval and tool details, tokens (input/output/cached/thinking), cost, latency (TTFT/total/stage), retries, stop reason, guardrail verdicts, feedback/judge scores.

> [!question]- Q20. Metrics vs logs vs traces?
> Metrics: aggregates for dashboards/alerts; logs: discrete event records and audit; traces: causal span trees showing where time/cost/errors occur across retrieval, LLM calls, tools, sub-agents. Link them with a trace ID.

> [!question]- Q21. What's the status of OpenTelemetry GenAI conventions?
> Per mid-2026 sources they're still in "Development" status (moved to a dedicated repo in June 2026, no tagged releases), so attribute names may change — pin instrumentation versions and isolate naming behind a thin layer.

> [!question]- Q22. How would you monitor quality in production?
> Sampled calibrated-judge scoring (same judge as offline), rule-based checks (schema, citation validity), implicit signals (edits, regenerations, escalations), explicit feedback, human review queues, business outcomes, drift detection, and golden probe canaries.

> [!question]- Q23. What dashboards do you build?
> Health, cost, quality, safety/security, retrieval, agents, and drift — with p95 latency, $/task, judge pass rates, guardrail triggers, tool errors, and cluster shifts.

> [!question]- Q24. Describe incident response for "answers got worse."
> Detect, triage type (quality/outage/safety/cost), contain (flag off, fallback), diagnose via traces and version diffs (prompt/model/index/tool/provider), fix and verify with evals, add regression tests, communicate, blameless post-mortem.

> [!question]- Q25. How do you keep telemetry privacy-safe?
> Redact PII/secrets at ingestion, RBAC, retention TTLs, deletion by user, separate environments, vendor DPAs, opt-in content capture, and sample full content only for flagged traces.

## Release management
> [!question]- Q26. What do you version in an LLM app?
> Prompts, model IDs/params, tool schemas, retrieval index/embedder/chunker, guardrail policies, datasets, judge prompts/models, config/flags, dependencies — and record the version tuple on every trace.

> [!question]- Q27. Shadow vs canary vs A/B?
> Shadow runs new version on live traffic without user exposure for offline comparison; canary exposes a small percent with monitoring and ramp; A/B randomizes exposure to measure business impact with statistics and guardrail metrics.

> [!question]- Q28. Outline a model migration playbook.
> Track deprecations; define migration evals; baseline; adapt prompts and parameters (don't just swap IDs — prune old scaffolding, re-estimate token budgets); shadow → canary → ramp; document differences; keep rollback until stable.

> [!question]- Q29. Pitfalls of A/B testing LLM features?
> Randomization unit leakage, shared caches/memory contaminating arms, SRM, underpowered tests, novelty effects, missing guardrail metrics, over-slicing, relying only on judge scores without business outcomes.

> [!question]- Q30. What should a CI gate check?
> Non-inferior primary metrics by CI, zero sentinel (must-never-fail) failures, safety ASR thresholds, latency/cost budgets, schema validity — plus artifacts and an override path with justification.

## Fine-tuning
> [!question]- Q31. When should you fine-tune?
> After prompting, RAG, and structured outputs plateau on behavior/format/style needs, for consistent house style or strict format, to shrink prompts/latency/cost via a smaller model, or for on-prem small models — with evals and enough quality data.

> [!question]- Q32. When shouldn't you?
> When the gap is missing/changing knowledge (use RAG), you lack evals or quality data, prompting hasn't been optimized, or maintenance cost across base-model upgrades outweighs benefits.

> [!question]- Q33. Why can fine-tuning on new facts increase hallucination?
> The model may learn to produce confident-sounding answers without grounding in knowledge it can't reliably store, encouraging guessing; facts are hard to update and uncited.

> [!question]- Q34. Explain LoRA.
> Freeze W and learn a low-rank update ΔW = (α/r)·B·A (B zero-initialized), training r(d+k) parameters instead of d·k; adapters can be merged or hot-swapped, enabling cheap multi-tenant customization.

> [!question]- Q35. What does QLoRA change?
> The frozen base is quantized to 4-bit (NF4, double quantization, paged optimizers) while adapters train in higher precision, cutting memory so larger models fit on one GPU.

> [!question]- Q36. SFT vs DPO vs RL fine-tuning?
> SFT teaches behavior from demonstrations; DPO optimizes preferences from chosen/rejected pairs (tone, safety, subtle quality) without RL infra; RL with verifiable/grader rewards improves reliability on checkable tasks but risks reward hacking and requires robust graders.

> [!question]- Q37. What is distillation and when is it valuable?
> Training a small student on a stronger teacher's (filtered) outputs/logits for a narrow, high-volume task to meet latency/cost targets; evaluate against the teacher on your test set and check provider terms for using outputs.

> [!question]- Q38. How do you evaluate a fine-tuned model fairly?
> Held-out test and shadow traffic, compare against well-optimized prompt baselines (equal effort) and larger models, check slices, regressions in general/safety capability, robustness out of distribution, seed variance, and evaluate the *served* (possibly quantized) artifact; include cost-benefit.

> [!question]- Q39. Common fine-tuning bugs?
> Wrong chat template, loss on prompt tokens, train/test leakage, noisy labels, overtraining, forgetting, weak prompt baselines, evaluating loss instead of task success, adapter merge errors, post-training quantization degradation.

## Serving
> [!question]- Q40. When does self-hosting beat APIs?
> High, steady utilization (or heavy batch workloads), strict data residency/air-gap needs, latency control, customization (multi-LoRA); idle GPUs, engineering cost, and redundancy needs often make APIs cheaper at low/spiky volume.

> [!question]- Q41. How do you size a deployment?
> Weights = params × bytes; KV cache per token × context × concurrency; add 10–20% overhead; fit to GPU memory (tensor parallel if needed); measure goodput under SLO with realistic prompts; add replicas for peak and N+1 redundancy; compute $/M tokens vs API.

> [!question]- Q42. Key vLLM features that matter in production?
> PagedAttention, continuous batching, chunked prefill, prefix caching, quantization support, speculative decoding, multi-LoRA, tensor/pipeline parallelism, OpenAI-compatible serving.

> [!question]- Q43. What is cache-aware routing?
> Routing requests with shared prompt prefixes to the replica that already holds their KV cache, raising hit rates and cutting TTFT/cost.

> [!question]- Q44. How do you handle overload?
> Admission control and bounded queues, `429/503` with `Retry-After`, priority-based load shedding, timeouts, circuit breakers, graceful degradation (smaller model, shorter context, cached answers, hosted fallback); avoid unbounded queues that collapse latency.

> [!question]- Q45. Why re-run quality evals after quantization or an engine change?
> Speed/memory changes can degrade accuracy in task-specific ways; you must evaluate the exact served artifact, not just benchmark speed.

## Guardrails & security
> [!question]- Q46. Describe a layered guardrail stack.
> Input (auth, limits, PII, injection/jailbreak classifiers, scope), context (trust labeling, ACL filtering, document scanning), model (system policy, safety-tuned models, structured outputs), output (safety/PII/policy/groundedness checks, sanitization), action (tool allow-lists, arg validation, approvals, budgets), operations (logging, anomaly detection, kill switch).

> [!question]- Q47. How do you evaluate guardrails?
> Labeled harmful/benign/near-miss/injection/PII sets (multilingual, obfuscated), measure per-category precision/recall, false-block rate on benign traffic, end-to-end attack success rate, latency/cost, and regression-test after changes.

> [!question]- Q48. Fail-open vs fail-closed?
> Fail-closed for high-risk checks (actions, PII leakage); fail-open with logging for low-risk content filters; decide per risk and availability needs.

> [!question]- Q49. Why treat model output as untrusted?
> It can carry injected content or malformed data leading to XSS, SQL/command injection, or unsafe tool calls; sanitize rendering, parameterize queries, validate arguments, and enforce least privilege.

> [!question]- Q50. What belongs in a red-team program?
> Threat model, manual and automated attacks (direct/indirect injection, jailbreaks, multilingual, multi-turn, tool abuse), scoring by ASR/severity with utility retention, fixes (architecture first), regression tests, recurring cadence, and residual-risk documentation.

> [!question]- Q51. System prompt leakage — how do you design for it?
> Assume prompts can be extracted; keep secrets, credentials, and authorization logic out of prompts; use canary tokens to detect leakage.

## Governance & FinOps
> [!question]- Q52. Provider vs deployer under the EU AI Act?
> Providers develop/place AI systems on the market (heavier obligations); deployers use them under their authority (oversight, monitoring, logs, informing people). Customers may become providers if they substantially modify or white-label — determine roles early with counsel.

> [!question]- Q53. What happened to the EU AI Act timeline in 2026 (per secondary sources)?
> A Digital Omnibus (Regulation (EU) 2026/1744) reportedly entered into force 27 July 2026, deferring high-risk obligations (Annex III to 2 Dec 2027; Annex I to 2 Aug 2028) while GPAI obligations (since Aug 2025) and Article 50 transparency (from Aug 2026, with a watermarking transition to Dec 2026) continue — verify with primary sources; still start inventory/classification now.

> [!question]- Q54. What goes into a system/model card?
> Purpose and scope, intended/out-of-scope uses, users, models and versions, data sources, evaluation methods/results with slices, limitations, safety measures, human oversight, monitoring, contacts.

> [!question]- Q55. How do you meet "right to delete" with LLM systems?
> Maintain data inventory and deletion workflows across prompts/logs/traces, vector stores, caches, fine-tuning datasets, and backups per policy; test end-to-end; avoid using personal data in training unless permitted.

> [!question]- Q56. NIST AI RMF in one line?
> A voluntary framework organized around Govern, Map, Measure, Manage functions for managing AI risk (with a Generative AI profile) — a common vocabulary for governance programs. (Verify current versions.)

> [!question]- Q57. Meaningful human oversight?
> Human-in-the-loop (approve actions), on-the-loop (monitor/intervene), in-command (set boundaries/shut down) — with reviewers who have time, context, and authority; avoid rubber-stamping and automation bias.

> [!question]- Q58. Cut a $50k/month LLM bill by 50% — your plan?
> Attribute spend by feature/tenant; compute cost per successful task; apply caching, trim context/tool output, shorten outputs, move offline work to Batch, route easy traffic to smaller models and lower effort (eval-validated), distill/fine-tune the highest-volume task, consider self-hosting if utilization justifies; set budgets/alerts.

## FDE scenarios
> [!question]- Q59. A customer asks, "How will we know it keeps working after launch?"
> Explain the quality flywheel: calibrated evals in CI, online sampled scoring, dashboards/SLOs, feedback → new test cases, canary releases with rollback, model-deprecation tracking, quarterly red-teams and governance reviews — and hand over runbooks and dashboards.

> [!question]- Q60. The customer's security team blocks launch pending "proof it's safe." What do you deliver?
> A governance pack: architecture/data-flow with trust boundaries, threat model and OWASP mapping, guardrail stack with measured precision/recall and red-team ASR, access control/tenancy design, data retention/deletion/residency, human-oversight design, evaluation report with slices and limitations, incident playbook, and vendor attestations — plus a staged rollout with monitoring.

Next → [[12-Resources-and-Practice-Links]]
