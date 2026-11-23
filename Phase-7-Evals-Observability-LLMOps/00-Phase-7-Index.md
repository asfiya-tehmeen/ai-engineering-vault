---
tags: [phase-7, index, evals, llmops]
---
# Phase 7 — Evals, Observability & LLMOps

> [!abstract] Why this phase exists
> Phases 4–6 taught you to **build** LLM systems. This phase teaches you to **run them reliably over time**: measure quality with trustworthy evals, see what's happening in production, ship changes safely, adapt models when prompting isn't enough, serve them efficiently, secure them, and keep them compliant. It's the difference between a demo and a product — and it's what customers pay FDEs to put in place. **Evals are the most under-rated skill in AI engineering**; if you master one thing in this vault, make it this.

**Duration:** 5–7 weeks · **Prereq:** Phases 4–6 (APIs, RAG, agents), Phase 2 (evaluation basics, PyTorch for fine-tuning).

> [!warning] Freshness & source quality
> Tooling and regulation move fast. In this phase: **OpenTelemetry's GenAI conventions are still "Development" status** (not stable) per mid-2026 sources; **EU AI Act deadlines were shifted by the 2026 "Digital Omnibus"** per secondary legal sources; **judge-calibration thresholds differ between blogs** (they're rules of thumb). My searches did not surface primary documentation for specific eval tools (Braintrust, Langfuse, Promptfoo, Inspect) or for ISO/IEC 42001 and NIST AI RMF, so those sections are written from established knowledge and flagged. **Verify dates, versions, and legal obligations against primary sources (and your counsel) before relying on them.**

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-Eval-Foundations]] | eval-driven development, error analysis first, datasets, graders, statistics, regression & CI, online vs offline |
| 02 | [[02-LLM-as-Judge-and-Eval-Methods]] | judge design, calibration & bias correction, pairwise, synthetic data, multi-turn/safety evals, contamination |
| 03 | [[03-Observability-and-Monitoring]] | logs/traces/metrics, OpenTelemetry GenAI, tooling, online evals, drift, feedback loops, incident response |
| 04 | [[04-Release-Management-and-Experimentation]] | prompt/config versioning, CI/CD, shadow/canary/A-B, model migration & deprecation, data flywheel |
| 05 | [[05-Fine-Tuning-and-Adaptation]] | when to fine-tune, SFT, LoRA/QLoRA, DPO, RL fine-tuning, distillation, evaluation, tooling |
| 06 | [[06-Serving-and-Infrastructure]] | managed vs self-host economics, vLLM/SGLang, GPU sizing, autoscaling, multi-LoRA, gateways, SLOs, load testing |
| 07 | [[07-Guardrails-and-Security-Hardening]] | guardrail layers, classifiers, PII, red-teaming, OWASP LLM Top 10, data protection |
| 08 | [[08-Governance-Compliance-and-FinOps]] | EU AI Act & GDPR/HIPAA/SOC 2, ISO 42001, NIST AI RMF, model cards, risk, vendor mgmt, FinOps |
| 09 | [[09-Build-an-Eval-and-LLMOps-Pipeline]] | eval harness, judge calibration, CI gate, tracing, prompt registry, canary routing, LoRA SFT sketch |
| 10 | [[10-Exercises]] | 90+ exercises |
| 11 | [[11-QA-Interview-Questions]] | 60 Q&A |
| 12 | [[12-Resources-and-Practice-Links]] | docs, papers, tools, courses |
| 13 | [[13-Projects-and-Checkpoint]] | 6 projects + self-assessment |

## Weekly plan (6 weeks)
| Week | Focus | Output |
|---|---|---|
| 1 | Notes 01, 02 | error analysis on real traces → taxonomy → first eval suite with code graders |
| 2 | Notes 02, 09 | calibrated judge (TPR/TNR, bias-corrected estimates), CI gate |
| 3 | Notes 03, 04 | tracing + dashboards, prompt registry, shadow/canary experiment |
| 4 | Note 05 | LoRA SFT (and DPO if time) with before/after evals; decision memo "fine-tune or not?" |
| 5 | Notes 06, 07 | serve a model with vLLM, load-test, size GPUs; guardrail layer + red-team results |
| 6 | Note 08 + capstone | governance pack + [[13-Projects-and-Checkpoint\|LLMOps capstone]] |

## Completion checklist
- [ ] Run an error-analysis session on ≥100 real traces and produce a failure taxonomy driving the eval design
- [ ] Build a golden set with dev/test split and code-based graders; report metrics with CIs
- [ ] Build an LLM judge, measure its **TPR/TNR on human labels**, and bias-correct its aggregate estimate
- [ ] Implement an eval CI gate that blocks regressions; explain flaky-eval handling
- [ ] Instrument an app with traces (prompt version, tokens, cost, latency) and build the dashboards you'd page on
- [ ] Version prompts/config as code; ship a change via shadow → canary → full rollout with rollback
- [ ] Execute a model migration playbook with measured parity
- [ ] Decide between prompting, RAG, fine-tuning, and distillation with evidence; run a LoRA SFT with proper evals
- [ ] Serve an open model (vLLM), load-test it, and produce a sizing/cost recommendation
- [ ] Implement layered guardrails and measure their precision/recall, latency, and false-block cost
- [ ] Produce a governance pack: risk assessment, model/system card, data-flow & retention, human-oversight design
- [ ] Capstone shipped on GitHub

Back to [[../00-Roadmap]].
