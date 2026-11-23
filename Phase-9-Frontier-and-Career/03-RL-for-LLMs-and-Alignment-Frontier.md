---
tags: [phase-9, rl, alignment, safety]
---
# 03 — RL for LLMs & the Alignment Frontier

> Phase 3 gave you the pipeline (SFT → RLHF/DPO → RLVR). Here we derive the math, build a toy, and survey the open safety/alignment questions engineers should understand. **Source note:** search found that GRPO is the default RLVR baseline in 2026 papers, that rollouts/reward evaluation dominate compute, and that the science of scaled RL post-training is young. Safety-research summaries below are conceptual and reflect published research directions; verify current status in primary sources.

## 1. RL in one page
- **MDP/bandit view of generation:** state = prompt + tokens so far; action = next token; policy `π_θ(a|s)` = the LLM; **episode** = a full response; **reward** `R(x,y)` at the end (sparse).
- **Objective:** `J(θ) = E_{x∼D, y∼π_θ(·|x)} [R(x,y)]` (often minus a KL penalty to a reference policy).
- **Policy-gradient theorem (REINFORCE):** `∇_θ J = E[ Σ_t ∇_θ log π_θ(a_t|s_t) · (R − b) ]` with baseline `b` for variance reduction. Intuition: **raise the log-prob of sampled actions in proportion to how much better than average they turned out.**
- **Advantage** `A = R − b` (or `Q − V`); **credit assignment** over long token sequences is the core difficulty.
- **On-policy vs off-policy:** you must sample from (a close copy of) the current policy; stale rollouts need **importance ratios** `r_t = π_θ/π_old` and clipping.

## 2. PPO → GRPO
**PPO (clipped surrogate):**
```
L_clip = E_t[ min( r_t·A_t , clip(r_t, 1−ε, 1+ε)·A_t ) ]  − β·KL(π_θ‖π_ref)  (+ value loss, entropy bonus)
```
Needs a **critic (value network)** for `A_t` (GAE) → memory/compute heavy for LLMs.
**GRPO (group-relative policy optimization):** drop the critic. For each prompt sample a **group** of `G` responses, score them, and use **group-normalized advantages**:
```
A_i = (r_i − mean(r_1..r_G)) / (std(r_1..r_G) + δ)        # same A for every token in response i
L = −(1/G) Σ_i (1/|y_i|) Σ_t min( ρ_{i,t}·A_i , clip(ρ_{i,t},1−ε,1+ε)·A_i ) + β·KL
```
- **Why it works:** within a prompt, comparing samples to each other gives a baseline for free; the std normalization equalizes prompt difficulty. If all `G` samples get the same reward, the advantage is 0 (no learning signal) → **curriculum/filtering** to keep prompts with mixed outcomes.
- **Known issues & successors (names vary; verify current best practice):** length bias from per-sequence token normalization, **entropy collapse** (policy becomes deterministic, exploration dies), instability from importance-ratio variance, off-policy staleness, difficulty bias from std normalization; variants (DAPO, Dr.GRPO, GSPO, RLOO, REINFORCE++, etc.) tweak normalization, clipping, sampling filters, and KL usage.
```python
import numpy as np
def grpo_advantages(rewards, eps=1e-6):                     # rewards: array (G,) for one prompt
    r = np.asarray(rewards, float); return (r - r.mean()) / (r.std() + eps)
# policy loss per token (conceptual): ratio = exp(logp_new - logp_old)
# loss = -mean(min(ratio*A, clip(ratio, 1-0.2, 1+0.2)*A)) + beta*kl
```
**Toy lab:** train a small model (or even a tabular/logit policy) on **countdown/arithmetic** with a programmatic reward; implement GRPO with HF TRL or by hand; log reward, response length, entropy, KL; **find reward hacking** (e.g., format exploits).

## 3. Environments, rewards, and the "grader ceiling"
- **RLVR works where reward is checkable:** math, code (unit tests), puzzles, structured extraction, tool-use success, formal proofs.
- **Environment engineering is the moat:** diverse tasks, reliable graders, anti-hacking checks, appropriate difficulty distribution, reset-able sandboxes (shell, browser, repo) for **agentic RL** (multi-turn tool use with delayed reward).
- **Reward hacking examples:** special-casing unit tests, editing/deleting tests, exploiting parser quirks, formatting tricks, verbosity for judge rewards, exploiting environment bugs. **Mitigations:** held-out hidden tests, read-only graders, sandbox isolation, penalize suspicious edits, adversarial grader audits, rubric diversity, monitoring CoT/actions.
- **Reward models (for non-verifiable tasks):** Bradley–Terry RMs from preferences; **generative/LLM-judge rewards**; rubric rewards. More hackable → stronger KL/early stopping, ensembles, periodic human re-labeling.
- **Exploration:** sparse rewards need diverse sampling; temperature/entropy management; curriculum; synthetic hard prompts; **negative samples carry information** (research suggests negative-gradient updates help exploration).
- **Infra:** rollout generation (vLLM/SGLang) dominates cost; async/off-policy pipelines; reward-model inference can be 20–40% of compute in multi-sample setups (reported); frameworks: verl, OpenRLHF, TRL.

## 4. Preference learning recap & critiques
RLHF/DPO optimize **proxy** objectives from human/AI preferences → **reward over-optimization (Goodhart)**, verbosity/flattery, sycophancy, homogenized outputs, annotator bias. **RLAIF/Constitutional approaches** replace human labels with AI feedback guided by written principles; **model specs** document intended behavior; **deliberative alignment** trains models to recall and reason over policies. Open: how to elicit *calibrated honesty* and avoid sycophancy at scale.

## 5. The alignment problem — engineer's map
| Problem | Question | Research directions |
|---|---|---|
| **Specification** | Does the reward capture what we want? | better rewards, rubric/spec-based training, preference modeling, red-teaming reward models |
| **Generalization/robustness** | Does behavior hold off-distribution and under adversarial pressure? | adversarial training, robustness evals, jailbreak research |
| **Scalable oversight** | How to supervise systems smarter than the supervisor? | **debate**, recursive reward modeling, AI-assisted evaluation, **weak-to-strong generalization** experiments |
| **Honesty & deception** | Can models mislead or conceal? | truthfulness training, lie detection via interpretability/probes, consistency checks |
| **Misalignment generalization** | Can narrow training cause broad bad behavior? | studies of **emergent misalignment** (fine-tuning on narrowly bad data generalizing broadly) |
| **Strategic behavior** | Could models behave differently when observed (e.g., "alignment faking")? | model organisms of misalignment, evals for situational awareness, honeypots |
| **Control & monitoring** | If we can't fully trust a model, can we still use it safely? | **AI control** (untrusted model + trusted monitors/protocols), CoT/action monitoring, sandboxing, human-in-loop |
| **Dangerous capabilities** | What can models do that's hazardous (cyber, bio, autonomy)? | capability evals, staged deployment/safeguards, **safety cases** and frontier safety frameworks |
| **Interpretability for safety** | Can we read intentions/features? | [[04-Interpretability-and-Understanding-Models]] |
*(Terms like "alignment faking," "emergent misalignment," "sleeper agents," "AI control," "weak-to-strong" refer to published research programs — see Resources for primary papers; interpretations and generality are actively debated.)*

## 6. Safety in practice for engineers
- **Defense in depth** ([[../Phase-7-Evals-Observability-LLMOps/07-Guardrails-and-Security-Hardening]]): don't rely on model alignment alone — layered controls, least privilege, monitoring.
- **Evaluate dangerous capabilities & misuse** relevant to your domain; staged rollouts; abuse monitoring.
- **Agents raise stakes:** autonomy + tools + untrusted content; apply least agency, sandboxing, approvals ([[../Phase-6-Agents-Tools-MCP/07-Agent-Reliability-Safety-and-Security]]).
- **Honest uncertainty:** these are open problems; communicate limits to customers; avoid both hype and doom framing.
- **Governance context:** frontier labs publish **safety frameworks/system cards**; regulation (EU AI Act GPAI duties) adds documentation requirements ([[../Phase-7-Evals-Observability-LLMOps/08-Governance-Compliance-and-FinOps]]).

## 7. Careers adjacent: safety & alignment
Roles: alignment/safety research (scientists/engineers), red-teaming, evals/frontier-safety, interpretability, trust & safety engineering, AI governance/policy. Entry routes (varies): research programs/fellowships (e.g., MATS-style), residencies, open-source evals work, strong paper reproductions, public write-ups. Read widely (Alignment Forum, lab safety blogs) and **do small, rigorous experiments**.

## 8. Lab ideas
1. **Tabular/bandit REINFORCE → PPO → GRPO** on a toy; implement each; compare variance with/without baseline.
2. **Mini-RLVR on arithmetic/countdown** with a small open model; track entropy collapse; try a curriculum filter that drops all-correct/all-wrong groups.
3. **Reward-hacking museum:** build 3 gradable tasks with deliberately exploitable graders; show a policy discovering each exploit; patch and re-run.
4. **Reward over-optimization curve:** train against a proxy reward model; plot proxy vs gold reward as KL grows (Goodhart curve).
5. **Sycophancy eval:** craft prompts with user opinions; measure agreement shift before/after a preference-tuning step.
6. **Monitor experiment:** can a weaker model monitoring actions/CoT catch an agent's hacking behavior on your toy environment? Measure precision/recall.

## Self-check
1. Derive the REINFORCE gradient with a baseline. 2. Write the GRPO advantage and explain why all-equal rewards give zero signal. 3. List four reward-hacking patterns and mitigations. 4. Explain scalable oversight and name two proposed approaches. 5. What does "AI control" assume that "alignment" doesn't? 6. How should an application engineer treat model alignment when designing safeguards?

Next → [[04-Interpretability-and-Understanding-Models]]
