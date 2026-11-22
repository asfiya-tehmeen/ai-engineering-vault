---
tags: [phase-3, post-training, rlhf, dpo, grpo]
---
# 04 — Post-Training & Alignment

> A pretrained "base model" completes text. Post-training turns it into an **assistant** that follows instructions, refuses harm, uses tools, and reasons. Most of what makes Claude/GPT/Gemini *feel* different lives here.

## Pipeline overview
```
Pretraining (base model)
  → [Mid-training: reasoning/code/long-context data]
  → SFT (supervised fine-tuning on demonstrations)
  → Preference optimization (RLHF / DPO / RLAIF)
  → RL with verifiable rewards (RLVR) for reasoning, coding, agentic tasks
  → Safety training, red-teaming, distillation, final polish
```

## 1. Supervised Fine-Tuning (SFT / instruction tuning)
- Train on `(prompt, ideal response)` pairs with the **same next-token loss**, but typically **loss-masked** to only the assistant tokens.
- Data: human-written demonstrations, curated/filtered model outputs, **synthetic data** (self-instruct, evol-instruct, distillation from stronger models), tool-use and agent traces, multi-turn chats, safety refusals.
- **LIMA hypothesis:** a small set of high-quality, diverse examples can teach *style/format*; knowledge comes from pretraining.
- Must use the **chat template** (see [[01-Tokenization]]). Packing, multi-turn masking, system prompts.
- Risks: **hallucination from teaching unknown facts** (training on facts the model doesn't know encourages guessing), over-refusal, style collapse, catastrophic forgetting. Use low LR (1e-5–2e-5 for full FT), 1–3 epochs.
- **PEFT:** LoRA/QLoRA (Phase 7) make SFT cheap.

## 2. Reward models & RLHF
**RLHF (Ouyang et al., InstructGPT 2022):**
1. Collect **preference data**: for a prompt, humans rank/compare 2+ responses.
2. Train a **reward model (RM)** `r_φ(x,y)` with the **Bradley–Terry** loss: `−log σ(r(x,y_w) − r(x,y_l))`.
3. Optimize the policy `π_θ` to maximize reward while staying close to the SFT reference `π_ref`:
   `max E[ r(x,y) ] − β·KL(π_θ ‖ π_ref)` using **PPO** (actor, critic/value, reference, reward model → memory-heavy).
- The **KL penalty** prevents reward hacking/drift into gibberish that fools the RM.
- **Failure modes:** **reward hacking** (exploit RM quirks: verbosity, flattery, formatting), **sycophancy** (agreeing with user's views), mode collapse/loss of diversity, length bias, annotator disagreement/bias.
- **RLAIF / Constitutional AI (Anthropic):** AI feedback guided by written principles replaces much human labeling; model critiques & revises its own outputs, then trains on preferences.
- **Process vs outcome reward models (PRM/ORM):** score reasoning steps vs final answers.
- **Generative / LLM-as-judge reward models**, rubric-based rewards for non-verifiable tasks (writing, helpfulness).

## 3. Direct preference methods (no explicit RM, no RL loop)
**DPO (Rafailov et al., 2023):** closed-form reparameterization turns RLHF into a classification loss on preference pairs:
```
L = −log σ( β [ log πθ(y_w|x)/π_ref(y_w|x) − log πθ(y_l|x)/π_ref(y_l|x) ] )
```
Simple, stable, cheap (two forward passes + reference log-probs). Variants: **IPO, KTO** (unpaired good/bad labels), **ORPO/SimPO** (reference-free), **cDPO, RPO**, online/iterative DPO (sample from current policy, re-label). Limits: offline data distribution mismatch; can reduce likelihood of both chosen and rejected; verbosity bias.
Rule of thumb: DPO-style for preference/style/tone and safety; **RL (on-policy)** for capabilities that need exploration (reasoning, coding, agents).

## 4. RL with Verifiable Rewards (RLVR) & reasoning models
**Idea:** for tasks with checkable answers (math, code with unit tests, puzzles, format constraints, some agentic tasks), the reward comes from a **programmatic verifier** — no learned RM to hack.
- **Policy-gradient with group baselines — GRPO (DeepSeekMath/DeepSeek-R1):** sample a **group** of `G` responses per prompt; reward each; compute **advantage = (r − mean_group)/std_group**; update with PPO-style clipped ratio + KL; **no value/critic network** → cheaper than PPO. Now the default open recipe, with many successors/variants (DAPO, Dr.GRPO, GSPO, RLOO, REINFORCE++, etc. — verify which is current).
- Rewards: correctness (+1/0), format rewards (reasoning in `<think>` tags), length penalties, unit-test pass rates, partial credit, LLM-judge for rubric tasks.
- **DeepSeek-R1 (Jan 2025)** showed long **chain-of-thought (CoT)** with self-verification, backtracking and "aha moments" can **emerge from outcome-only RL** (R1-Zero), then a multi-stage pipeline (cold-start SFT → RL → rejection sampling SFT → RL) for readability and general ability. Triggered the open reasoning-model wave.
- **What RL changes:** more thinking tokens, self-correction, better search, better *reliability*; debated how much is new capability vs eliciting/sharpening what pretraining already contains. One small controlled 2026 study found gains were largely from learned *termination/format behavior* — a single-model result, treat as a caution not a verdict.
- **Challenges:** rollout cost dominates (generation is the bottleneck → vLLM/SGLang in the loop, verl/TRL/OpenRLHF frameworks), entropy collapse, reward hacking (e.g., special-casing test cases, exploiting environment bugs), sparse rewards, length explosion, off-policy staleness, verifier coverage (non-verifiable domains).
- **Environments:** RL gym-style task suites (math, code, browser, terminal, SWE tasks) — **environment/data quality is the new moat**. Agentic RL trains multi-turn tool use against real/simulated environments (Phase 6).
- **Test-time compute scaling:** more thinking tokens → better answers (up to diminishing returns; reported gains can flatten at large budgets). Techniques: long CoT, **best-of-N / majority vote (self-consistency)**, verifier-guided search, tree search, budget forcing, adaptive thinking/effort controls exposed by APIs (Phase 4).

## 5. Distillation
Train a smaller **student** on a stronger **teacher's** outputs (SFT on reasoning traces — "R1-distill" style) or logits (KL). Cheap way to give small models reasoning ability; also how synthetic data flows between labs (check provider ToS about training on outputs).
- **On-policy distillation:** student samples, teacher scores per-token — efficient and avoids exposure bias.

## 6. Safety, alignment & behavior shaping
- **HHH** (helpful, honest, harmless) framing; **refusal training**, **red-teaming** (human + automated), **jailbreak** robustness, **adversarial training**.
- **Spec/constitution/model-spec documents** define intended behavior (OpenAI Model Spec, Anthropic's constitution). **Deliberative alignment**: train the model to recall & reason over written policies.
- **Refusal vs over-refusal trade-off.** Calibrated uncertainty & honesty (admit not knowing).
- **Sycophancy, deception, sandbagging, alignment faking, reward hacking, emergent misalignment (narrow bad fine-tuning generalizes broadly)** — active research; matters for deploying agents.
- **Safeguards beyond the model:** classifiers on inputs/outputs, system-level policy, rate limits, monitoring, human review (Phase 7).
- **Interpretability & control:** steering vectors, circuit analysis, chain-of-thought monitoring (Phase 9).

## 7. Tool use, structured output & agentic post-training
Models are trained on **function-calling formats**, JSON/grammar adherence, multi-step tool trajectories, computer-use, long-horizon coding. This is why API features like tool use and structured outputs work (Phase 4/6). Training on agent traces + RL in realistic environments is the main driver of 2025–26 agent improvements.

## 8. Practical: do it yourself (Hugging Face TRL)
```python
# SFT
from trl import SFTTrainer, SFTConfig
trainer = SFTTrainer(model="Qwen/Qwen2.5-0.5B", train_dataset=ds, args=SFTConfig(output_dir="out", assistant_only_loss=True))
# DPO
from trl import DPOTrainer, DPOConfig
# GRPO with a custom reward function
from trl import GRPOTrainer, GRPOConfig
def reward_correct(completions, answer, **kw): return [1.0 if extract(c) == a else 0.0 for c, a in zip(completions, answer)]
```
(Check TRL docs for current argument names — the API evolves.) Toy RLVR tasks: arithmetic, countdown, GSM8K-subset, unit-test code tasks.

## 9. Evaluating post-trained models
Arena-style preference (LMArena), instruction-following (IFEval), math/code (AIME, LiveCodeBench, SWE-bench), agent benchmarks, safety (HarmBench), human evals. Beware **Goodharting on public benchmarks** and **contamination**. Your *own* task-specific evals matter most (Phase 7).

## Self-check
1. Write the Bradley–Terry loss. 2. What does the KL term in RLHF do? 3. Derive intuition for DPO's implicit reward `β log π/π_ref`. 4. Why is GRPO cheaper than PPO? 5. Why verifiable rewards? Where do they fail? 6. What is reward hacking—give 3 examples. 7. SFT vs RL: which teaches *what*?

Next → [[05-Inference-and-Decoding]]
