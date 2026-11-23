---
tags: [phase-9, projects, checkpoint, final]
---
# 12 — Projects & Final Checkpoint

> Frontier projects signal **depth**; career projects signal **readiness**. Publish all of them: public repos, short write-ups, and (where useful) demos. For each: **hypothesis → setup → results with uncertainty → limitations → code**.

## Project 1 — `tts-lab`: test-time compute study *(Note 01)*
Pick a task with a checkable answer (math subset, code with tests, SQL result equality, structured extraction). Implement single-sample, majority-vote, best-of-N with (a) oracle, (b) noisy LLM verifier, (c) process-guided search, plus adaptive stopping and difficulty routing. Produce **accuracy–cost–latency frontiers** and a policy recommendation for a real route in your capstone.
**Deliverable:** repo + 3-page report; highlight where the noisy-verifier curve peaks and why.

## Project 2 — `quant-distill-lab`: efficiency study *(Note 02)*
Quantize an open 1–8B model with ≥3 methods; evaluate on **your task suite** (JSON validity, long-context, tool calls, multilingual, safety) plus perplexity; measure memory/tok/s with vLLM; analyze outlier layers; run a **distillation/QAD-style recovery** experiment and a roofline prediction vs measurement.
**Deliverable:** reproducible benchmark repo + "which config would you ship, and why" memo with cost model.

## Project 3 — `grpo-mini`: RLVR with reward-hacking audit *(Note 03)*
Train a small model on a verifiable task (countdown/arithmetic/unit-test tasks) with GRPO. Log reward/length/entropy/KL; implement the zero-variance-group filter; build a **reward-hacking museum** (3 exploitable graders, patched versions); chart the proxy-vs-gold over-optimization curve.
**Deliverable:** repo + blog post with curves and failure stories.

## Project 4 — `interp-lab`: small-model mechanistic study *(Note 04)*
On GPT-2 small (or your tinyformer): logit lens, induction-head analysis and ablations, activation patching heatmaps, a steering vector, and a **trained SAE** with causal validation (ablation/steering) and blind-scored labels. Include a "skeptic's checklist" applied to your own findings.
**Deliverable:** notebook + write-up with honest limitations.

## Project 5 — `paper-replication`: reproduce and extend *(Note 06)*
Choose one paper (any area) with a testable small-scale claim. Reproduce the core effect, run ablations under equal budgets, add one extension, and publish a **replication write-up** (arXiv-note/blog style) with code. Submit to maintainers/community for feedback; contribute a fix/doc PR if you found issues.
**Deliverable:** public repo + write-up + PR/issue.

## Project 6 (Capstone) — `career-launch-kit` + integrated capstone *(Notes 07, 08)*
1. **Integrated capstone** `enterprise-ai-teammate` (see [[08-Capstone-and-12-Month-Plan]]) with a frontier extension from Projects 1–4.
2. **Public presence:** personal site with 3 project pages, 3 write-ups, a 5-minute demo video, polished GitHub profile/READMEs.
3. **Application kit:** role-targeted resumes (×2), LinkedIn, target company list (30) with contacts, outreach templates, tracker with funnel metrics.
4. **Interview kit:** story bank (10 STAR+Learning), project walkthroughs (×3, 10 min each), system-design cheat sheets (RAG/agent/evals/serving/FDE), recorded mock loops (≥4) with feedback logs.
5. **Plan:** 12-month roadmap with quarterly gates; weekly operating rhythm; learning OS populated (reading-log ≥12 entries, experiment-log ≥8, 2 "state-of" docs).
**Acceptance:** an independent reviewer (mentor/peer) can run your capstone, read your results, and judge you ready for the target role.

---

# 🏁 Final Whole-Roadmap Mastery Assessment

Rate each 0–3 (0 = never done, 1 = understand, 2 = can do with references, 3 = can teach/ship). **Target: ≥2 everywhere for your target role; ≥3 in your chosen specialty.** Revisit every quarter.

### Phase 1 — Software foundations
| Skill | 0–3 |
|---|---|
| Idiomatic Python, typing, async, packaging, testing | |
| DSA at medium level; complexity analysis | |
| Git/CLI/Linux; HTTP/REST/auth; SQL (CTEs/windows/indexes) | |
| Docker, CI, secrets, basic cloud | |
### Phase 2 — ML & deep learning
| Math for ML (linear algebra, calculus, probability, info theory) | |
| Classical ML pipelines, metrics, leakage, calibration | |
| Backprop by hand; PyTorch training loops; debugging | |
| CNN/RNN/embeddings/generative-model concepts | |
### Phase 3 — LLM internals
| Tokenization, attention, architecture variants (GQA/MLA/RoPE/MoE) | |
| Pretraining/scaling math; post-training (SFT/RLHF/DPO/GRPO) | |
| Inference: KV cache, decoding, batching, quantization, cost/latency estimation | |
| Built and trained a GPT; understand limits/hallucinations | |
### Phase 4 — APIs
| Messages API mechanics, errors/retries, usage/cost, current param changes | |
| Prompting, structured outputs, tool use loop, streaming, async/batch | |
| Context engineering, prompt caching, multimodal/doc pipelines | |
### Phase 5 — RAG
| Embeddings, vector search, hybrid + rerank, chunking/ingestion/permissions | |
| RAG evals (golden sets, metrics, pitfalls), advanced RAG (GraphRAG/agentic) | |
### Phase 6 — Agents
| Agent loops, tool design, MCP, memory/state, multi-agent trade-offs | |
| Agent security (trifecta, sandboxing, approvals) and evals (pass^k) | |
### Phase 7 — Evals/LLMOps
| Error analysis, datasets, graders, calibrated judges, statistics | |
| Observability, release management, model migration | |
| Fine-tuning decisions (LoRA/DPO/RFT/distill), serving & sizing, guardrails, governance/FinOps | |
### Phase 8 — FDE
| Discovery, decomposition, scoping, ROI, risk management | |
| Rapid prototyping/demos; enterprise integration (SSO/SCIM/legacy/network/security reviews) | |
| Communication/change management; production readiness/handoff | |
| Interview-ready: decomposition, debugging, role-plays, STAR stories | |
### Phase 9 — Frontier & career
| Test-time compute, RLVR/GRPO, reward hacking, safety/alignment concepts | |
| Quantization/distillation/MoE/hybrid architectures, roofline reasoning | |
| Interpretability tools and skepticism | |
| Reading/reproducing papers; maintaining a learning OS | |
| Portfolio, resume, networking, applications, offers | |

## Final capstone review rubric (peer/mentor scores 1–5)
Problem framing & scope · Engineering quality · Evals & statistics · Security & privacy · Cost/latency awareness · Production readiness & ops · Customer-facing artifacts · Communication/writing · Frontier extension rigor · Honesty about limitations.

## Where to go from here
- **Specialize** (pick 1–2): evals/LLMOps · agents & security · RAG/knowledge systems · inference/efficiency · FDE/solutions · safety/interpretability · AI product.
- **Teach** what you know: write, present, mentor — it's the fastest way to deepen mastery.
- **Revisit this vault yearly:** update provider/tool specifics (APIs, models, regulations), refresh projects, and add your own field notes and case studies.
- **Remember the principle that runs through every phase:** *measure, don't guess; build the simplest thing that works; design for failure; keep humans appropriately in the loop; communicate honestly.*

🎉 **You've reached the end of the roadmap.** Back to [[00-Phase-9-Index]] · [[../00-Roadmap]]
