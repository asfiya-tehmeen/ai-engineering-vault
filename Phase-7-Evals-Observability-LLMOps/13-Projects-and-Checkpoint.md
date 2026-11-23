---
tags: [phase-7, projects, checkpoint]
---
# 13 — Projects & Phase Checkpoint

> Each repo: README with **Problem & users · System diagram · Dataset(s) (provenance, versions, splits) · Metrics with CIs · Judge calibration report (TPR/TNR/κ) · Cost & latency · Failure taxonomy and what you fixed · Security/privacy notes · Runbook**. In interviews, **showing a failure taxonomy you derived from real traces and the measured improvement it drove** is worth more than any architecture diagram.

## Project 1 — `evalkit`: reusable evaluation framework *(Notes 01, 02, 09)*
Datasets (JSONL + versioning + hash), graders (regex/schema/state/execution), multi-trial runner with caching and concurrency, **calibrated LLM judges** with TPR/TNR/κ storage and **bias-corrected estimates with two-stage bootstrap CIs**, paired comparison reports, pass^k, sliced dashboards (HTML), cost/latency tracking, CI gate action.
**Stretch:** pluggable adapters for apps/agents; judge-drift monitor with a frozen calibration set; synthetic-data generator with human filter UI.

## Project 2 — `trace-lab`: observability & incident response stack *(Notes 03, 04)*
Instrument a RAG or agent app (from Phases 5–6) with OTel traces + structured logs; version tuple on every trace; PII redaction/retention; Langfuse/Phoenix (or Grafana) dashboards for health/cost/quality/safety/retrieval/agents/drift; **online evals** (10% sampling + human review queue); golden-probe canaries; SLOs and alerts.
**Game day:** inject 3 regressions (prompt change, provider/model behavior change, retrieval index bug); diagnose from traces; write blameless post-mortems and add regression tests.

## Project 3 — `ship-it`: release pipeline with shadow/canary/rollback *(Note 04)*
Config-driven router (prompt/model/effort/fallback), sticky canary assignment, shadow mode with offline comparison, A/B experiment harness (SRM check, power calc, guardrail metrics), automated rollback on metric breach, model-deprecation tracker, and a **model-migration report** comparing two models on quality/cost/latency with prompt adaptation notes.

## Project 4 — `tune-or-not`: fine-tuning decision lab *(Note 05)*
One real task (e.g., support-reply policy compliance, structured extraction, or text-to-SQL). Baselines: optimized prompt on a small and a large model, RAG-only. Then **LoRA SFT** (and optionally DPO/GRPO), distillation to a smaller model, quantization. Evaluate on a held-out test + slices + forgetting/safety regressions; compute break-even volume and maintenance costs.
**Deliverable:** a decision memo with numbers (including the case where the answer is "don't fine-tune") and the reproducible training/eval repo.

## Project 5 — `serve-bench`: self-hosting sizing & gateway *(Note 06)*
Serve an open model with vLLM; realistic load tests (length distributions, streaming, prefix sharing) via GuideLLM/GenAI-Perf; compare quantization/prefix caching/chunked prefill/speculative decoding on **speed and eval quality**; build an **LLM gateway** (LiteLLM or custom) with auth, budgets, fallback to a hosted API, caching, logging; autoscaling/overload chaos tests; **API vs self-host break-even model** for three load profiles.

## Project 6 (Capstone) — `llmops-in-a-box`: governed, evaluated, observable deployment *(everything)*
Take a RAG or agent application (your Phase 5/6 capstone) and make it **production-ready for a regulated customer**:
- **Evals:** failure taxonomy from ≥100 traces; ≥150-case suite with sentinels; code graders + ≥2 calibrated judges; paired-CI CI gate; nightly full run; pass^k for agent flows.
- **Observability:** OTel traces, dashboards, online scoring, SLOs/alerts, cost attribution by tenant.
- **Release:** versioned prompts/config, shadow → canary → ramp, tested rollback, deprecation tracker.
- **Adaptation:** a documented fine-tune/distill/route decision (implemented if justified).
- **Serving/gateway:** gateway with auth/budgets/fallback; optionally self-hosted open-weight path for sensitive data.
- **Guardrails/security:** layered stack with measured precision/recall and false-block rate; red-team report (≥100 attacks, ASR before/after); OWASP LLM/Agentic mapping; secrets/PII handling; deletion workflow.
- **Governance pack:** system card, data-flow diagram, risk assessment/DPIA-style doc, EU-AI-Act role/tier note (with "verify with counsel" caveats), human-oversight design, incident playbook, vendor appendix.
- **Customer-facing artifacts:** exec summary with metrics and ROI model; ops runbook; 5-minute demo including a regression caught by CI and a canary rolled back.
This is the strongest FDE portfolio piece: it proves you can take an AI system from prototype to **trusted production**.

## Phase 7 Self-assessment (0–3; need ≥2 before Phase 8)
| Skill | 0–3 |
|---|---|
| Run error analysis and build a failure taxonomy from real traces | |
| Build datasets with splits, sentinels, versioning, and coverage mapping | |
| Compute CIs, paired comparisons, and power; avoid Goodhart/leakage | |
| Write code graders and binary judges with rubrics | |
| Calibrate a judge (TPR/TNR/κ) and bias-correct aggregate estimates | |
| Detect/mitigate judge biases and drift | |
| Design multi-turn, safety, and synthetic evals | |
| Build an eval CI gate and handle flaky evals | |
| Instrument tracing/logging with version tuples, tokens, cost | |
| Build dashboards/SLOs and online evals; detect drift | |
| Run an incident response and write a blameless post-mortem | |
| Version prompts/config; ship via shadow/canary/A-B with rollback | |
| Execute a model migration/deprecation playbook | |
| Decide prompt vs RAG vs fine-tune vs distill with evidence | |
| Run LoRA/QLoRA SFT (and explain DPO/RFT) and evaluate fairly | |
| Size, serve, and load-test an open model; compute break-even | |
| Build a gateway with auth, budgets, fallback, caching | |
| Implement a layered guardrail stack and evaluate it | |
| Run a red-team and map to OWASP; harden output handling and secrets | |
| Produce a governance pack (system card, data flows, risk assessment) and a cost-per-task model | |

## Mock interview (60 min)
1. 10 min — "Your RAG assistant's quality dropped last week. Walk me through diagnosis." (traces, version tuples, drift, provider change, retrieval index, evals)
2. 15 min — "Design the eval system for a customer-support agent." (taxonomy, datasets, graders, judges + calibration, pass^k, CI, online monitoring)
3. 10 min — whiteboard: bias-corrected pass-rate estimate and CI given judge TPR/TNR.
4. 10 min — "Should we fine-tune?" (ladder, data, baselines, forgetting, ROI, maintenance)
5. 15 min — scenario: "The CISO needs proof this is safe; the DPO asks about the EU AI Act and deletion rights." (governance pack, guardrails + red-team evidence, data flows, roles, caveats, staged rollout)

## What you've unlocked
You can make LLM systems **measurable, observable, releasable, adaptable, secure, and governable** — the discipline that separates prototypes from products. Phase 8 puts it all in the context of the **Forward Deployed Engineer role**: discovery, scoping, rapid prototyping, enterprise integration, stakeholder communication, delivery and handoff, case studies, and interview preparation.

⏸️ **Pause point.** Tell me when you're ready for **Phase 8 — Forward Deployed Engineering** (I'll research current FDE job descriptions and interview loops at AI labs and enterprise AI companies, common engagement patterns, and the soft-skill and delivery frameworks first).

Back to [[00-Phase-7-Index]] · [[../00-Roadmap]]
