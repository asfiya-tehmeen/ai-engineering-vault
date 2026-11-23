---
tags: [phase-7, exercises]
---
# 10 — Exercises (Phase 7)

🟢 easy · 🟡 medium · 🔴 hard. **Cost/safety rules:** cap spend per experiment, use small models and Batch/caching for large eval runs, never train on or log real customer data without rights/redaction, and run load tests only against infrastructure you own. Keep everything in `phase7-exercises/` with a results table (config → metric ± CI, cost, latency) and **save raw outputs** so results are reproducible.

## A. Eval foundations (Note 01)
- [ ] 🟢 A1. Take any LLM feature you built in Phases 4–6. Collect 100 inputs (real or realistic). Read every output and write one-line failure notes. Cluster into a **failure taxonomy** with counts.
- [ ] 🟢 A2. For three failure categories, decide the grader type (code / judge / human) and justify.
- [ ] 🟡 A3. Build a dataset v1 (JSONL) with ≥60 cases, tags, expectations, a `sentinel` subset, and a dev/test split; write a changelog and hash.
- [ ] 🟡 A4. Implement code graders (regex/schema/state checks) and run N=3 trials per case; report pass rate, **pass^3**, cost, latency p50/p95 by tag.
- [ ] 🟡 A5. Statistics drill: given n=120, p=0.78, compute the 95% CI analytically and by bootstrap; simulate how often a true 3-point improvement is detected at n=100/300/1000 (power curve).
- [ ] 🟡 A6. Paired vs unpaired comparison: compare two prompts on the same cases with both methods; show the paired test's higher power.
- [ ] 🟡 A7. Make a flaky eval deterministic: identify variance sources (sampling, retrieval, tools, judge); reduce with trials, caching, pinned versions; show variance before/after.
- [ ] 🔴 A8. **Goodhart experiment:** optimize a prompt against a small dev set until it overfits; show the dev–test gap; fix with held-out test and refresh.
- [ ] 🔴 A9. Build a **coverage map** linking failure-taxonomy categories to test cases; find uncovered categories; add cases.
- [ ] 🔴 A10. Contamination audit: check your eval set for leakage into prompts/few-shots/fine-tuning data/retrieval index; add canary strings; document findings.
- [ ] 🔴 A11. Write an **eval charter** (1 page) for a customer-facing assistant: goals, metrics, owners, cadence, gates, how SMEs participate.

## B. LLM-as-judge & eval methods (Note 02)
- [ ] 🟢 B1. Write a binary judge prompt with rubric, 2 pass and 2 fail examples; require evidence-before-verdict in structured output.
- [ ] 🟡 B2. **Calibrate:** hand-label 100 outputs (50 dev/50 test), compute TPR, TNR, agreement, Cohen's κ; iterate the prompt on dev; report test results.
- [ ] 🟡 B3. Implement the **Rogan–Gladen correction** and a two-stage bootstrap CI; compare raw judge pass rate vs corrected rate vs human ground truth.
- [ ] 🟡 B4. Bias tests: (a) position bias in pairwise (swap order), (b) verbosity bias (pad answers), (c) self-preference (same vs different model family as judge). Quantify each.
- [ ] 🟡 B5. Compare 1–5 Likert vs binary+critique judges on the same data for reliability (agreement with humans, test–retest consistency).
- [ ] 🟡 B6. Pairwise protocol: A vs B with randomized order, both orders, tie handling; compute win rate ± CI; fit Bradley–Terry over 3 variants.
- [ ] 🔴 B7. **Judge drift study:** run the same judge prompt across two model versions (or two providers) on a frozen calibration set; measure shifts; design an alarm.
- [ ] 🔴 B8. Cheap-judge cascade: small judge screens, strong judge adjudicates disagreements/uncertain cases; evaluate cost vs agreement loss.
- [ ] 🔴 B9. Synthetic eval generation: structured grid (persona × intent × difficulty × edge case) → LLM generates → human filter; compare performance on synthetic vs real test cases and quantify optimism bias.
- [ ] 🔴 B10. Multi-turn eval with a **simulated user**; validate simulator consistency across seeds; measure conversation-level success and variance.
- [ ] 🔴 B11. Safety eval: build a 100-case benign/borderline/harmful set; measure refusal precision/recall and over-refusal; add multilingual/obfuscated variants.

## C. Observability (Note 03)
- [ ] 🟢 C1. Add structured logging to an LLM call: request ID, model, prompt version, tokens, cost, latency, stop reason, retries.
- [ ] 🟡 C2. Add OpenTelemetry tracing (manual spans) to a RAG or agent app; view traces in Langfuse/Phoenix/Jaeger; identify the slowest stage.
- [ ] 🟡 C3. Build the **seven dashboards** (health, cost, quality, safety, retrieval, agents, drift) in Grafana/Langfuse/your tool of choice with sample data.
- [ ] 🟡 C4. Implement **online evaluation**: 10% sampling + calibrated judge + human-review queue; show scores by route and version.
- [ ] 🟡 C5. Drift detection: embed weekly query samples, cluster, alert on new clusters and distribution shift (PSI/KS); simulate drift and show detection time.
- [ ] 🟡 C6. Golden-probe canary: scheduled synthetic requests with expectations; detect a silent regression you inject (e.g., changed tool schema).
- [ ] 🔴 C7. PII-safe telemetry: redact PII/secrets at ingestion; prove with tests; implement retention TTL and per-user deletion across traces.
- [ ] 🔴 C8. Define SLOs and error budgets for an LLM feature; simulate budget burn and the policy response.
- [ ] 🔴 C9. **Incident drill:** inject a regression (bad prompt version / provider change / retrieval index bug); diagnose from traces only; write a blameless post-mortem and add regression tests.
- [ ] 🔴 C10. Cost attribution: tag requests by tenant/feature; produce showback report and detect a runaway-agent anomaly automatically.

## D. Release management (Note 04)
- [ ] 🟢 D1. Move prompts into files with versions and render tests; add `served_by` version tuple to every trace.
- [ ] 🟡 D2. Build a config-driven router (model, effort, fallback, prompt version) with typed config validation.
- [ ] 🟡 D3. CI pipeline: unit/contract tests → smoke eval → comparison report on PR with paired CI → gate.
- [ ] 🟡 D4. Implement sticky **canary** assignment (hash of user ID) and compare canary vs stable on online metrics.
- [ ] 🟡 D5. **Shadow launch:** run a new model/prompt in parallel on live-like traffic; judge/compare offline; produce a go/no-go.
- [ ] 🔴 D6. **Model migration exercise:** migrate a feature from model A to B: baseline, adapt prompts (prune cruft, adjust params), re-estimate token budgets, run evals, shadow, canary; write a migration report incl. cost/latency.
- [ ] 🔴 D7. A/B test design doc: unit of randomization, SRM checks, primary/guardrail metrics, power calc, stopping rule; run a simulated test with injected effect and detect it.
- [ ] 🔴 D8. Rollback test: automate a flag flip to previous prompt/model/index; measure time-to-rollback in a game day.
- [ ] 🔴 D9. Deprecation tracker: inventory model IDs across services, map to provider EOL dates, auto-open tickets 90/60/30 days before.

## E. Fine-tuning & adaptation (Note 05)
- [ ] 🟢 E1. Write the **"fine-tune or not?" decision memo** for two scenarios (policy-compliant tone for support replies; answering from changing product docs). Defend each choice.
- [ ] 🟡 E2. Build an SFT dataset (300–1000 examples) from corrected traces; verify chat template, loss masking (print tokens + labels), dedupe, PII scrub, leakage check vs test.
- [ ] 🟡 E3. LoRA SFT a 1–3B model (Unsloth/TRL); track val loss **and** task metrics; compare against the best prompt baseline on a base model and on a bigger API model.
- [ ] 🟡 E4. LoRA hyperparameter sweep: rank {4,16,64}, target modules (attention-only vs all-linear), LR; plot quality vs trainable params.
- [ ] 🟡 E5. Forgetting check: evaluate base vs tuned on a general-capability mini-suite and your safety suite; quantify regressions.
- [ ] 🔴 E6. **DPO** after SFT on preference pairs (from judge-labeled or rejection-sampled outputs); evaluate tone/format/safety changes and verbosity drift.
- [ ] 🔴 E7. **RL with a verifiable grader (GRPO/RFT-style)** on a structured-extraction task: train with a programmatic reward; hunt for reward hacking; evaluate on held-out data.
- [ ] 🔴 E8. **Distillation:** generate teacher outputs (filtered by graders), SFT a small student, compare cost/latency/quality to teacher; check provider ToS on using outputs.
- [ ] 🔴 E9. Fine-tune an **embedding model** or **reranker** for a retrieval task (Phase 5); compare ROI with fine-tuning the generator.
- [ ] 🔴 E10. Quantize the tuned model and serve it; re-run evals **on the served artifact** to confirm no unacceptable degradation.
- [ ] 🔴 E11. Multi-LoRA: train 3 tenant adapters; serve on one base via vLLM; measure per-adapter quality, switching overhead, and throughput.

## F. Serving & infrastructure (Note 06)
- [ ] 🟢 F1. Write a **sizing calculator** (weights, KV cache per token/sequence, overhead, GPU fit, tokens/s estimate, $/M tokens); validate against a real model's `config.json`.
- [ ] 🟡 F2. Serve a small open model with vLLM (or Ollama); expose an OpenAI-compatible endpoint; call it from your app via the same client interface as a hosted API.
- [ ] 🟡 F3. **Load test** with realistic prompt-length distributions and streaming; plot TTFT/TPOT/throughput vs concurrency; find the knee; compute goodput at an SLO.
- [ ] 🟡 F4. Compare configs: quantization (bf16 vs FP8/AWQ), prefix caching on/off, chunked prefill, speculative decoding; report speed **and** eval quality.
- [ ] 🟡 F5. Build an **LLM gateway** (LiteLLM or custom) with auth, per-key budgets, retries/fallback between self-hosted and API, logging, exact-match caching.
- [ ] 🔴 F6. Autoscaling: deploy on Kubernetes (kind/minikube with CPU stand-in, or a cloud GPU) with HPA/KEDA on queue depth; test cold-start time; add warm replicas.
- [ ] 🔴 F7. Overload & chaos tests: kill replicas, flood with long prompts; verify admission control, `429/503` + `Retry-After`, graceful degradation, no latency collapse.
- [ ] 🔴 F8. **Break-even analysis:** API vs self-hosted for three load profiles (steady, spiky, batch); include utilization, redundancy, and engineering cost; recommend.
- [ ] 🔴 F9. Cache-aware routing: implement prefix-hash routing across 2 replicas; measure hit-rate and TTFT gains for shared-prefix workloads.
- [ ] 🔴 F10. Air-gapped dry run: assemble an offline bundle (container, weights, wheels), deploy without internet, document update/rollback procedure.

## G. Guardrails & security (Note 07)
- [ ] 🟢 G1. Draw your app's guardrail stack (input/context/model/output/action) and list the control + owner + failure mode at each layer.
- [ ] 🟡 G2. Implement PII detection/redaction (Presidio) before logging and before sending to an API; evaluate precision/recall on a labeled set.
- [ ] 🟡 G3. Add an injection/jailbreak classifier + an LLM-guard; measure precision/recall, false-block rate on benign traffic, latency; tune thresholds.
- [ ] 🟡 G4. Output hardening: sanitize rendering (escape HTML, strip images/links), validate tool args, parameterize SQL; test with malicious outputs.
- [ ] 🟡 G5. Denial-of-wallet defenses: per-user token/tool budgets, input caps, anomaly alerts; simulate an abuser.
- [ ] 🔴 G6. **Red-team sprint:** 100+ attack cases (direct, indirect via docs/web/tool outputs, multilingual, obfuscated, multi-turn); report ASR by category before/after fixes and utility retention.
- [ ] 🔴 G7. Secrets & prompt-leak drill: plant canary tokens in system prompt/context; attempt extraction; show detection; remove secrets from prompts.
- [ ] 🔴 G8. Supply-chain check: scan dependencies/containers, verify model checksum, convert a pickle weight to safetensors, review a third-party MCP server/skill.
- [ ] 🔴 G9. Map your app to the **OWASP LLM Top 10**: threat → control → test → residual risk table.
- [ ] 🔴 G10. Write a prompt-injection incident playbook and run a tabletop exercise.

## H. Governance, compliance & FinOps (Note 08)
- [ ] 🟢 H1. Classify 6 example systems by EU AI Act tier (e.g., resume screener, support chatbot, medical triage, code assistant, deepfake generator, internal summarizer) and your role (provider/deployer); note what's uncertain for counsel.
- [ ] 🟡 H2. Write a **system card** for one of your projects (purpose, users, data, models/versions, eval results with slices, limitations, oversight, monitoring).
- [ ] 🟡 H3. Produce a **data-flow diagram** with trust boundaries, data categories, regions, retention, subprocessors; identify GDPR/HIPAA issues.
- [ ] 🟡 H4. Implement a **right-to-delete** workflow across logs/traces, vector store, caches, fine-tuning data; test end-to-end.
- [ ] 🟡 H5. Build a **cost-per-successful-task** model for a feature with levers (cache, batch, routing, effort); validate with a pilot's usage logs; produce a monthly forecast with scenarios.
- [ ] 🔴 H6. **Fairness slice analysis:** counterfactual prompt pairs swapping sensitive attributes; measure outcome/quality disparities; propose mitigations and monitoring.
- [ ] 🔴 H7. Draft an **AI governance pack** (12 sections from Note 08) for a fictional bank deployment; get a peer to play CISO/DPO and critique it.
- [ ] 🔴 H8. Vendor-risk assessment of two model providers (data policies, ZDR/BAA availability, regions, deprecation policy, SLAs, indemnities, exit plan) → recommendation memo.
- [ ] 🔴 H9. Map your controls to **NIST AI RMF** functions (Govern/Map/Measure/Manage) and, optionally, ISO/IEC 42001 clauses; list gaps.

## I. Mini-capstones (pick 2)
- [ ] 🔴 I1. **Eval-as-a-service:** a reusable repo any LLM app can plug into (datasets, graders, calibrated judges, CI gate, reports).
- [ ] 🔴 I2. **LLM gateway + observability stack:** auth, budgets, routing, caching, tracing, dashboards, guardrails, chaos-tested.
- [ ] 🔴 I3. **Fine-tune vs prompt bake-off** on a real task with decision memo and cost model.
- [ ] 🔴 I4. **Governance-ready deployment:** a RAG/agent app with system card, threat model, red-team report, DPIA-style assessment, and monitoring runbook.

## Cadence
3–4 exercises/week; every week do one **transcript/trace reading hour** and update your failure taxonomy; review [[11-QA-Interview-Questions]] on Sundays.
