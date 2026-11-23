---
tags: [phase-9, exercises]
---
# 09 — Exercises (Phase 9)

🟢 easy · 🟡 medium · 🔴 hard. Frontier exercises are **small, rigorous experiments** — hypothesis first, fixed seeds, multiple runs, CIs, honest write-up (including failures). Career exercises produce **real artifacts** you'll use. Keep code in `phase9-exercises/` and log each experiment in your vault's experiment log ([[06-Reading-Papers-and-Doing-Research]] §8).

## A. Reasoning & test-time compute (Note 01)
- [ ] 🟢 A1. Implement `pass_at_k` (unbiased estimator) and unit-test it against brute-force enumeration on small cases.
- [ ] 🟢 A2. Take 50 GSM8K/MATH-style problems; sample N=16 answers from a small model; compute single-sample accuracy, majority@k, and oracle best-of-k for k∈{1,2,4,8,16}. Plot.
- [ ] 🟡 A3. Add a **noisy verifier** (LLM-judge or small reward model) and plot best-of-k accuracy vs k; find where it peaks and explain why (verifier exploitation).
- [ ] 🟡 A4. **Cost–accuracy frontier:** compare (a) one long-thinking call, (b) k short samples + vote, (c) cheap-model-then-escalate routing; plot accuracy vs $ and vs latency; recommend a policy.
- [ ] 🟡 A5. **Adaptive stopping:** sample until agreement ≥ threshold or budget hit; compare tokens used vs fixed-k for equal accuracy.
- [ ] 🟡 A6. Measure **overthinking:** run an easy-question set with low vs high reasoning effort; compare accuracy/cost/latency/format errors.
- [ ] 🔴 A7. **Budget forcing** with an open reasoning model: truncate/extend thinking with "Wait"; plot accuracy vs thinking tokens; compare to natural stopping.
- [ ] 🔴 A8. **Process-reward-guided search:** beam search over reasoning steps using a PRM or judge on a small math set; compare to best-of-N at equal compute.
- [ ] 🔴 A9. **Faithfulness probe:** insert a biasing hint in prompts; measure how often the answer changes and how often the stated reasoning mentions the hint.
- [ ] 🔴 A10. Reproduce the **inference-budget confound:** evaluate two models at equal vs unequal token budgets; show rank reversal; write 1 page on reporting standards.

## B. Efficiency: quantization, distillation, MoE, architectures (Note 02)
- [ ] 🟢 B1. Roofline calculator: given GPU peak FLOPs/bandwidth, compute ridge point and classify prefill vs decode for a 7B model; predict decode tok/s for bf16/int8/int4 and compare with measured vLLM.
- [ ] 🟢 B2. NumPy quantization study: per-tensor vs per-channel vs per-group INT8/INT4 on random + outlier-injected weight matrices; plot reconstruction error vs group size.
- [ ] 🟡 B3. Quantize a 1–8B model with ≥3 methods (e.g., bnb-NF4, AWQ/GPTQ, GGUF); compare memory, tok/s, perplexity **and** your task suite (JSON validity, long-context, tool calls).
- [ ] 🟡 B4. Find the **outlier channels** in a real model layer's activations; show they dominate error under naive W8A8; apply SmoothQuant-style scaling and re-measure.
- [ ] 🟡 B5. KV-cache quantization: measure long-context accuracy and memory with fp16 vs int8/fp8 KV.
- [ ] 🟡 B6. **Distillation:** teacher→student for a narrow task using (a) response-level SFT, (b) logit KL, (c) on-policy; compare cost/quality.
- [ ] 🔴 B7. **QAD toy:** 4-bit-quantize a small model; recover with KL distillation from the bf16 teacher vs QAT with the original loss vs PTQ; report accuracy recovered per training-token.
- [ ] 🔴 B8. **MoE from scratch:** top-2 router + load-balancing loss in tinyformer; plot expert utilization with/without aux loss; measure active vs total params and throughput.
- [ ] 🔴 B9. **Linear vs softmax attention** on copy/needle tasks; implement a small hybrid (few attention layers + linear layers) and measure recall vs speed/memory at long lengths.
- [ ] 🔴 B10. Implement **online softmax / tiled attention** in PyTorch (or Triton) and verify numerics; profile memory vs naive attention.
- [ ] 🔴 B11. Write a Triton kernel for a fused op (e.g., softmax or RMSNorm) and benchmark vs PyTorch; document what you learned about memory-bound kernels.

## C. RL for LLMs & alignment (Note 03)
- [ ] 🟢 C1. Derive the REINFORCE gradient with a baseline on paper; verify numerically on a 2-armed bandit.
- [ ] 🟡 C2. Implement REINFORCE → PPO-clip → GRPO on a toy sequence task (e.g., generate a sequence summing to a target); compare variance and sample efficiency.
- [ ] 🟡 C3. GRPO advantage unit tests: all-equal rewards → zero advantages; scaling invariance; effect of std normalization on easy vs hard prompts.
- [ ] 🔴 C4. **Mini-RLVR:** train a small open model on countdown/arithmetic with a programmatic reward (TRL GRPO or custom); log reward, length, entropy, KL; add a curriculum filter dropping groups with zero variance; compare learning curves.
- [ ] 🔴 C5. **Reward-hacking museum:** build 3 tasks with exploitable graders; show the exploit emerging; patch (hidden tests, read-only grader, penalties) and re-run.
- [ ] 🔴 C6. **Over-optimization curve:** optimize against a proxy reward model; plot proxy vs held-out "gold" reward vs KL; identify the Goodhart peak.
- [ ] 🔴 C7. **Sycophancy eval:** build 100 prompts with stated user opinions; measure agreement shift across models and before/after preference-tuning on your data.
- [ ] 🔴 C8. **Monitor experiment:** can a weaker model monitoring actions/CoT detect hacking in your toy environment? Report precision/recall and failure cases.
- [ ] 🔴 C9. Read 3 alignment papers (e.g., debate, weak-to-strong, AI control); write a 2-page critical synthesis: assumptions, evidence, what an application engineer should take away.

## D. Interpretability (Note 04)
- [ ] 🟢 D1. Logit lens on GPT-2 small: plot the correct token's rank by layer for 10 factual prompts; note where predictions "form."
- [ ] 🟡 D2. Induction heads: compute induction scores; ablate top heads; measure loss change on repeated random sequences.
- [ ] 🟡 D3. Probing with control tasks: linear probes by layer for a syntactic/semantic property; compare to random-label control (probe capacity) and to patching results.
- [ ] 🟡 D4. Activation patching heatmap on an IOI-style or factual-recall prompt pair; identify the causally important components.
- [ ] 🟡 D5. Steering vector: contrastive mean-difference direction (e.g., formal vs casual); sweep strength; measure effect and side effects on unrelated tasks.
- [ ] 🔴 D6. **Train an SAE** on a GPT-2 layer (or your tinyformer); report variance explained, L0, loss recovered, dead features; label 20 features by top examples and **blind-score** a peer's labels for validity.
- [ ] 🔴 D7. Causal validation: ablate/steer 5 SAE features and test whether predicted behavior changes; compute a random-direction baseline.
- [ ] 🔴 D8. Reproduce the induction-head **phase change** during training of a 2-layer attention-only model; plot loss and induction score over steps.
- [ ] 🔴 D9. Write a skeptic's checklist for interpretability claims and apply it to one published result; produce a short critique.

## E. Data, continual learning & open problems (Note 05)
- [ ] 🟡 E1. **Model collapse demo:** repeatedly fine-tune a small model on its own samples; track diversity (distinct n-grams, entropy) and quality over generations; test mitigation by mixing real data.
- [ ] 🟡 E2. **Forgetting experiment:** fine-tune on task B after task A; measure A's drop; test replay buffer, LoRA adapters, and regularization.
- [ ] 🟡 E3. **Data quality filter:** train a quality classifier on a labeled subset; filter a corpus; train two small models (filtered vs raw) at equal tokens; compare.
- [ ] 🔴 E4. **Memory system study:** compare context-only, compaction, file memory, and vector memory on a 100-turn synthetic assistant task with recall probes and poisoning attempts.
- [ ] 🔴 E5. **Test-time training toy:** implement a small fast-weight/memory layer updated at inference on the context; compare long-context recall to a baseline.
- [ ] 🔴 E6. Create a **"state of X" doc** (e.g., state of agent security) with sources, confidence ratings, dates, and "what would change my mind"; share for critique.
- [ ] 🔴 E7. Pick three open problems from the list; for each, write a 1-page experiment proposal (hypothesis, minimal setup, metrics, risks).

## F. Reading papers & research practice (Note 06)
- [ ] 🟢 F1. Set up the learning OS in your vault (reading-log, experiment-log, state-of, flashcards, weekly-review, questions) and complete 4 weekly reviews.
- [ ] 🟡 F2. Do three-pass reads on 6 papers (one per area: reasoning, RL, efficiency, interp, RAG/agents, evals); write summaries; compare your read with a published critique/blog.
- [ ] 🟡 F3. **Reproduction:** reproduce a small result from a paper (qualitative effect), with seeds/CIs and a README; compare to the paper's claim.
- [ ] 🔴 F4. **Replication write-up** (blog/arXiv note style): claim, setup, results, ablations, limitations, what differed; publish and invite feedback.
- [ ] 🔴 F5. **Benchmark audit:** read 30 tasks + graders of a public eval; document flaws; propose and test a fix; share with maintainers.
- [ ] 🔴 F6. Contribute a **merged PR** to an OSS project in the ecosystem (eval harness, serving, interpretability tooling, docs/cookbook).
- [ ] 🔴 F7. Present a paper (30 min) to a reading group/peers; handle Q&A; refine.

## G. Career strategy (Note 07)
- [ ] 🟢 G1. Write a **target-role memo:** primary + secondary roles, why, required skills, gaps, 90-day plan.
- [ ] 🟢 G2. Audit your portfolio against the "hireable project" checklist; list concrete fixes (metrics, README, CI, demo).
- [ ] 🟡 G3. Build/refresh your **GitHub profile README**, pin 3–5 repos, and add architecture diagrams + results tables to flagship READMEs.
- [ ] 🟡 G4. Create a **personal site** with project pages (problem → approach → results → lessons) and one blog post.
- [ ] 🟡 G5. Write **resume v1** with impact bullets (method + metric + scale); get 3 reviews (recruiter/hiring manager/peer); iterate; make role-specific variants.
- [ ] 🟡 G6. Build a **target list** of 30 companies with 3 contacts each; draft 5 outreach messages; send 10; track responses.
- [ ] 🟡 G7. Do **10 informational chats**; write a summary of insights and what you changed afterward.
- [ ] 🟡 G8. Set up a **job-search tracker** with funnel metrics; after 3 weeks, diagnose the weakest stage and adjust.
- [ ] 🔴 G9. Run **4 full mock loops** (AI engineer ×2, FDE ×2) with structured feedback; track improvement across rounds.
- [ ] 🔴 G10. Build an **offer decision matrix** and rehearse a negotiation (recorded) using a hypothetical competing offer.
- [ ] 🔴 G11. Write a **"first 90 days" plan** for a specific target company.
- [ ] 🔴 G12. Publish **three public artifacts** (write-ups/videos/PRs) in 8 weeks; measure engagement and conversations generated.

## H. Capstone & 12-month plan (Note 08)
- [ ] 🟡 H1. Write your **capstone spec** (domain, scope, components per phase, acceptance criteria) and get feedback.
- [ ] 🟡 H2. Create your **12-month plan** with quarterly gates; adjust for your starting point and constraints.
- [ ] 🔴 H3. Build the capstone's **first vertical slice** (data → retrieval → answer → eval → CI) in 2 weeks and demo it.
- [ ] 🔴 H4. Complete a **quarterly review** (re-assess skills, mock interview, portfolio audit, gap plan).

## Cadence
Weekly: one frontier experiment or reading deep-dive, one career action (outreach/application/mock), one public artifact in progress. Review [[10-QA-Interview-Questions]] on Sundays.
