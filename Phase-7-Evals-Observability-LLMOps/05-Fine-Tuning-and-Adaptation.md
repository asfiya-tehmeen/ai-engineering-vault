---
tags: [phase-7, fine-tuning, lora, dpo, distillation]
---
# 05 — Fine-Tuning & Model Adaptation

> Fine-tuning is a powerful tool that is **overused as a first resort**. The consistent 2026 practitioner advice: **exhaust cheaper options first — prompt → RAG → fine-tune → distill** — and fine-tune to change **behavior/format/style/latency-cost**, not to inject **facts**. (Post-training theory: [[../Phase-3-Transformers-LLM-Internals/04-Post-Training-and-Alignment]].)

## 1. The adaptation ladder (decision framework)
| Step | Try when | Cost | Typical limits |
|---|---|---|---|
| **1. Better prompting & context engineering** | always first | very low | ceiling on strict format/consistency; token cost of long prompts |
| **2. RAG / tools** | missing/changing knowledge, citations, permissions | low–medium | retrieval errors; doesn't change style/behavior |
| **3. Structured outputs / constrained decoding** | format reliability | low | shape only |
| **4. Fine-tune (SFT / preference / RFT)** | consistent style/format, domain behavior, tool-use habits, high-volume cost/latency reduction, small-model quality | medium–high (data + training + eval + maintenance) | needs data & evals; model-version lock-in; facts go stale |
| **5. Distillation** | make a small/cheap model match a large one on a *narrow* task | medium | task-specific; needs teacher data & eval |
| **6. Continued pretraining / from scratch** | new language/domain at scale; sovereign needs | very high | rarely warranted |
**Signals you should fine-tune:** prompting plateaus (e.g., 95% where you need 99.5% on strict structure/policy behavior); prompts are huge and costly (fine-tune bakes behavior in); latency/cost need a smaller model; consistent house style/tone; specialized tool-calling or domain jargon; on-prem small model requirement. **Signals you shouldn't:** facts change often; <~100 quality examples (rule of thumb, not law); no eval set; unclear failure mode; prompting hasn't been seriously optimized.

## 2. What fine-tuning can and can't do
- **Good at:** format/schema adherence, tone/persona, classification/extraction patterns, domain vocabulary usage, multi-step procedures, reducing prompt length, small-model specialization, safety/refusal style.
- **Poor at:** reliably adding *new factual knowledge* (hard to learn, hard to update, can encourage hallucination — fine-tuning on facts the base model doesn't know can teach it to guess), bypassing base-model capability ceilings, replacing retrieval/citations.
- **Risks:** catastrophic forgetting/regression on general skills, overfitting to narrow data, data leakage into eval, safety regressions, license/data-rights issues, maintenance burden (re-tune per base-model upgrade).

## 3. Data is the project
- **Quality > quantity:** hundreds to a few thousand *excellent*, diverse, correct examples often beat tens of thousands of noisy ones (LIMA-style findings for style/format).
- **Sources:** curated production traces (human-corrected), SME-authored demonstrations, **synthetic data** from a stronger model (verify!), rejection-sampled outputs filtered by graders/tests, public datasets (check licenses).
- **Formatting:** use the **exact chat template/tokenizer** of the base model; mask loss to assistant tokens; include system prompts as in production; multi-turn and tool-call traces if needed.
- **Cleaning:** dedupe, remove PII/secrets, fix label noise, balance classes/intents, include **hard negatives and refusal/abstention cases**, cover edge conditions; check for **contamination with eval sets**.
- **Splits:** train/val/test with the test set frozen and never used for training or prompt-tuning.
- **Rights & privacy:** ensure you may train on the data (customer contracts, GDPR), strip sensitive content, track provenance; check provider ToS about training on model outputs.

## 4. Methods
### 4.1 Full fine-tuning
Update all weights. Highest capacity, highest cost (memory ≈ 16–20 bytes/param with Adam in mixed precision), risk of forgetting, one full copy per variant. Rarely needed for app teams.
### 4.2 Parameter-efficient fine-tuning (PEFT)
- **LoRA (Low-Rank Adaptation):** freeze `W ∈ ℝ^{d×k}`; learn low-rank update `ΔW = (α/r)·B·A` with `B ∈ ℝ^{d×r}`, `A ∈ ℝ^{r×k}`, rank `r ≪ min(d,k)` (e.g., 8–64). Trainable params `r(d+k)` instead of `dk` → ~0.1–2% of the model. `B` initialized to zero so training starts at the base model. Merge `ΔW` into `W` for zero-overhead inference or keep adapters separate and **hot-swap per tenant/task** (multi-LoRA serving).
  - Hyperparameters: rank `r`, `α` (scaling), dropout, **target modules** (attention projections q,k,v,o and often MLP up/gate/down — applying to all linear layers usually helps), learning rate (higher than full FT, e.g., ~1e-4–2e-4), epochs (1–3), batch size, warmup, weight decay.
  - Findings reported in the literature: LoRA often matches full FT on narrow tasks, **learns less and forgets less** on out-of-domain skills; capacity limits appear for large-data/new-knowledge settings.
- **QLoRA:** base model quantized to **4-bit (NF4)** with double quantization + paged optimizers; train LoRA adapters in higher precision. Makes 30–70B-class fine-tunes possible on a single large GPU; trades memory for slightly slower training/setup complexity.
- **Variants:** DoRA (magnitude/direction decomposition), LoRA+ (different LRs for A/B), rsLoRA, VeRA, AdaLoRA, prefix/prompt tuning, adapters (bottleneck), IA³. Treat as optimizations after a LoRA baseline.
### 4.3 Supervised fine-tuning (SFT)
Next-token loss on assistant responses (loss-masked). Learns format/behavior from demonstrations. Epochs 1–3; watch val loss and **task evals** (loss ≠ task success).
### 4.4 Preference optimization
- **DPO** (and IPO/KTO/ORPO/SimPO variants): train on (prompt, chosen, rejected) pairs without a reward model/RL loop; good for tone, safety, verbosity, subtle quality preferences. Typically run **after SFT**. Needs good preference data (human, AI-labeled with a calibrated judge, or from rejection sampling by graders). Beta (KL strength) is a key hyperparameter; watch for verbosity drift and likelihood displacement.
- **RLHF with PPO:** heavier infra; mostly labs.
### 4.5 RL fine-tuning with verifiable/grader rewards (RFT / RLVR / GRPO)
For tasks with programmatic or model-based graders (structured extraction correctness, tool-use success, code tests, policy compliance), RL against the grader can improve reliability beyond SFT. Providers offer **managed reinforcement fine-tuning** (e.g., OpenAI's RFT — details vary; check docs) and open stacks (TRL GRPO, verl, OpenRLHF). Risks: **reward hacking** (exploit grader quirks), sparse rewards, entropy collapse, rollout cost, grader quality = ceiling. Invest in grader robustness and held-out evals.
### 4.6 Distillation
Train a small **student** on a larger **teacher's** outputs: (a) **response distillation** (SFT on teacher generations, possibly filtered by graders/tests), (b) **logit/KL distillation** when you can access teacher probabilities (open models), (c) **on-policy distillation** (student samples, teacher grades per token). Great for narrow, high-volume tasks (classification, extraction, routing) to hit latency/cost targets. Verify provider terms about using outputs to train models. Always evaluate student vs teacher on *your* test set and slices.
### 4.7 Other adaptation
- **Embedding/reranker fine-tuning** (often the best ROI for retrieval systems — Phase 5).
- **Continued pretraining** on domain text (rare; needs lots of data/compute).
- **Model merging** (task arithmetic, TIES, SLERP) to combine adapters/models — experimental, evaluate carefully.
- **Test-time adaptation** via in-context examples (dynamic few-shot retrieval) — cheaper than FT for many cases.

## 5. Hosted vs self-managed fine-tuning
| | Hosted FT APIs (providers/clouds) | Self-managed (open models) |
|---|---|---|
| Effort | minimal infra; upload data, get endpoint | you manage GPUs, training code, serving |
| Control | limited hyperparameters, model choice per provider; some top models not tunable | full control, any open model, any method |
| Data | leaves your boundary (check retention/ZDR/terms) | stays in your environment |
| Cost | per-token training + (often higher) inference price | GPU hours + engineering time |
| Lock-in | tied to provider/base version | portable weights (watch license) |
| When | quick wins, small teams | privacy/on-prem, cost at scale, research flexibility |
Availability differs widely by provider/model (some frontier models can't be fine-tuned at all) — verify current docs. Cloud platforms (Bedrock/Vertex/Azure) offer managed tuning for select models.

## 6. Tooling (verify current docs)
- **Hugging Face TRL** (SFT, DPO, GRPO, reward modeling), **PEFT** (LoRA/QLoRA), **Transformers**, **Accelerate**, **bitsandbytes** (4/8-bit), **datasets**.
- **Unsloth** (fast, memory-efficient single-GPU LoRA/QLoRA), **Axolotl** (YAML-driven training), **torchtune** (PyTorch-native), **LLaMA-Factory**, **Lightning**, **DeepSpeed/FSDP** for multi-GPU, **verl/OpenRLHF** for RL.
- **Experiment tracking:** W&B, MLflow, TensorBoard. **Serving adapters:** vLLM multi-LoRA, LoRAX, SGLang LoRA support.
- Compute: Colab/Kaggle (small), Modal/RunPod/Lambda/Vast (rentals), cloud (SageMaker, Vertex), on-prem GPUs.

## 7. Training recipe (LoRA SFT, practical defaults — tune on your data)
1. Pick base model (size/license/capabilities; **instruct** variant for chat tasks; start small, e.g., 1–8B, to iterate fast).
2. Build dataset in the model's chat format; verify tokenization and loss masks on a few examples by printing tokens+labels.
3. LoRA on all linear layers, `r=16–64`, `α=r` to `2r`, dropout 0–0.1, LR ~1e-4–2e-4 (cosine, warmup 3–5%), 1–3 epochs, effective batch 16–64 sequences (grad accumulation), max length covering your data, bf16.
4. **Evaluate during training** on val loss *and* a small task-eval subset; early-stop on task metrics.
5. Compare against **baselines**: best prompt on the base model, best prompt on a larger model, RAG-only.
6. Check **regressions** (general ability, safety, other tasks) and **slices**.
7. Merge or serve adapter; load-test; canary.
8. Plan **retraining** cadence and base-model upgrade path.

## 8. Evaluating a fine-tuned model (don't skip)
- Held-out test set and **real-traffic shadow**; compare to prompting baselines **at equal effort** (fair prompt optimization for baselines!).
- Metrics: task success, format validity, policy compliance, hallucination/faithfulness, safety/ASR, over-refusal, latency/cost.
- **Capability regression suite** (general reasoning, instruction following, multilingual, tool use) and **forgetting checks**.
- **Distribution shift:** performance on inputs outside training distribution.
- **Variance across seeds** (train 2–3 seeds if cheap); report CIs.
- **Cost-benefit:** training+serving+maintenance vs savings/quality gain; break-even volume.

## 9. Common failure modes
Wrong chat template or tokenization · loss on prompt tokens · data leakage between train/test · training on noisy/incorrect labels · overtraining (memorization, brittle outputs) · catastrophic forgetting · teaching hallucination via unknown facts · reward hacking in RL · comparing against a weak baseline prompt · eval only on loss · merging adapters incorrectly · quantization after training degrading quality (evaluate the *served* artifact) · license violations.

## 10. Decision memo template ("fine-tune or not?")
Problem & metric · baseline results (prompt, RAG, larger model) · failure analysis (what exactly fails; is it behavior or knowledge?) · data availability & rights · expected gain & cost (train, serve, maintain) · risk (forgetting, safety, lock-in) · plan (data, method, eval, rollout) · go/no-go criteria.

## Self-check
1. When should you fine-tune vs RAG vs prompt? 2. Explain LoRA math and why B is zero-initialized. 3. QLoRA in one paragraph. 4. DPO vs RFT: when each? 5. Why can fine-tuning on facts increase hallucination? 6. Design the evaluation for a fine-tuned extraction model.

Next → [[06-Serving-and-Infrastructure]]
