---
tags: [phase-9, reasoning, test-time-compute]
---
# 01 — Reasoning Models & Test-Time Compute

> **Source note:** 2026 commentary agrees that **inference-time scaling + RLVR** is the most visible shift in how frontier systems improve, and that RL post-training compute is large and its "science" is young (one 2026 paper cites individual ablation runs costing tens of thousands of GPU-hours). Beyond that, specifics are contested. This note teaches the **mechanisms** so you can evaluate any new claim. (Foundations: [[../Phase-3-Transformers-LLM-Internals/04-Post-Training-and-Alignment]], [[../Phase-3-Transformers-LLM-Internals/06-Context-Capabilities-and-Limits]].)

## 1. The big idea
Pretraining scaling spends compute **once, before deployment**. **Test-time (inference-time) compute** spends it **per query**: let the model generate more tokens, sample more candidates, or search before answering. If accuracy rises predictably with test-time compute, you gain a **second scaling axis** (alongside params, data, pretraining compute) — and a **cost/latency dial per request** (the `effort`/thinking controls you used in Phase 4).

## 2. Where test-time compute comes from (taxonomy)
| Family | Mechanism | Examples |
|---|---|---|
| **Sequential (longer thinking)** | one long chain of thought with self-checks, backtracking | reasoning models trained with RL; "budget forcing" (append "Wait" to extend / truncate to stop) |
| **Parallel (many samples)** | sample N independent solutions; pick one | **majority vote / self-consistency**, best-of-N with a verifier/reward model |
| **Search** | explore a tree of partial solutions with value guidance | beam search with process reward models, tree/graph-of-thoughts, MCTS-style methods |
| **Iterative refinement** | generate → critique/execute → revise | self-refine, code with unit-test feedback, evaluator–optimizer loops |
| **Tool-augmented** | use code/search/calculators during reasoning | agents ([[../Phase-6-Agents-Tools-MCP/00-Phase-6-Index|Phase 6]]) |
| **Adaptive compute** | spend more only on hard problems | difficulty routers, effort levels, early stopping |

## 3. How reasoning models are trained (recap + intuition)
- **Base model** with strong priors → optional **cold-start SFT** on reasoning traces → **RL with verifiable rewards (RLVR)** on math/code/logic/tool tasks (e.g., GRPO-family algorithms) → rejection-sampling SFT/distillation → safety/helpfulness polish.
- **What RL changes:** increases the probability of reasoning behaviors that lead to verifiably correct answers — longer deliberation, self-verification, backtracking, decomposition. **Debated:** how much is *new capability* vs *eliciting/sharpening* what pretraining already contains; one small 2026 controlled study reported much of its gain came from learned *termination/format behavior* (single study — a caution, not a verdict).
- **Reward types:** outcome reward (final answer correct), **process reward** (step correctness; needs step labels or learned PRMs), format rewards, length penalties, unit-test pass rates, rubric/LLM-judge rewards for non-verifiable tasks (noisier, hackable).
- **Distillation** transfers reasoning traces to small models ([[../Phase-7-Evals-Observability-LLMOps/05-Fine-Tuning-and-Adaptation]]).

## 4. Scaling behavior & limits (what to expect)
- **Gains are real but concave:** accuracy vs thinking tokens/samples typically rises then **flattens**. Reports of "virtually no gain from 16k→32k thinking tokens" on hard math suggest models don't always *use* extra budget productively; research on "learning to explore" aims to extrapolate.
- **Best-of-N is bounded by verifier quality:** with a perfect verifier, pass@N → coverage; with a noisy reward model, **reward hacking** appears as N grows (the verifier gets exploited) — gains peak then fall.
- **Majority vote needs an answer space where agreement ≈ correctness** (math with unique answers works; open-ended text doesn't).
- **Overthinking:** extra reasoning can hurt easy tasks (wasted cost, sometimes worse answers or format drift). Use routing/effort control.
- **Cost reality:** reasoning tokens are billed as output tokens, add latency, and increase rate-limit pressure (providers have imposed tighter limits on heavy users). Evaluate **accuracy per dollar and per second**, not accuracy alone ([[../Phase-4-Building-with-LLM-APIs/05-Context-Engineering-Caching-Cost]]).
- **Evaluation confound:** benchmark scores depend heavily on **inference budget** (tokens, samples, tools). Comparisons without equal budgets are meaningless — a 2026 paper explicitly studies how inference compute shapes frontier evaluation. Always report budget with score.
- **Pass@k vs pass^k** matter again: reliable agents need consistency ([[../Phase-6-Agents-Tools-MCP/08-Agent-Evaluation-and-Observability]]).

## 5. Verifiers & reward models (the crux of parallel/search methods)
| Verifier | Strength | Weakness |
|---|---|---|
| **Programmatic** (tests, exact answer, theorem prover, SQL result equality) | exact, unhackable-ish | only for checkable tasks |
| **Outcome reward model (ORM)** | cheap, trained on correctness | sparse signal; can be fooled |
| **Process reward model (PRM)** | step-level credit; better search guidance | needs step labels or noisy synthetic labels; step segmentation issues |
| **LLM judge with rubric** | flexible | biases, drift, calibration ([[../Phase-7-Evals-Observability-LLMOps/02-LLM-as-Judge-and-Eval-Methods]]) |
| **Self-verification** | free | models are poorly calibrated about their own errors; verification can be easier than generation but isn't guaranteed |
**Principle:** *verification is often easier than generation* — that asymmetry is why sample-and-verify works. Invest in verifiers/environments; "the grader is the ceiling."

## 6. Long-horizon capability & agents
Frontier measurement trend: **task time horizons** — the length (in human-expert time) of tasks an agent can complete at a given reliability (e.g., 50% success). Reports show horizons growing over successive models; interpret cautiously (task selection, scaffold, reliability threshold). Related 2026 work explores **long-horizon scaling returns to compute** and **no-CoT time horizons** (how much a model can do without visible reasoning). Practical takeaway for engineers: **reliability at 80–99% matters more than 50% success**; decompose tasks, verify steps, checkpoint, and measure pass^k.

## 7. Faithfulness, monitorability & safety angle
- Displayed chain-of-thought may **not faithfully reflect** the true computation; models can omit influences (e.g., hints) from their stated reasoning.
- Yet CoT is a **useful monitoring surface** (it often reveals intent like reward hacking). Research argues for preserving **CoT monitorability** (avoid training pressure that teaches models to hide reasoning).
- For deployed systems: monitor **actions and outcomes**, not just reasoning text ([[../Phase-6-Agents-Tools-MCP/07-Agent-Reliability-Safety-and-Security]]); many APIs return only summaries or encrypted reasoning.

## 8. Hands-on: build a test-time compute lab
```python
import numpy as np, collections

def majority_vote(answers):
    c = collections.Counter(a for a in answers if a is not None)
    return c.most_common(1)[0][0] if c else None

def best_of_n(cands, scorer):                       # scorer: verifier/reward fn returning float
    return max(cands, key=scorer)

def pass_at_k(n, c, k):                              # unbiased pass@k from n samples with c correct
    if n - c < k: return 1.0
    return 1.0 - np.prod(1.0 - k / np.arange(n - c + 1, n + 1))

# Experiment loop (pseudo): for each problem, sample N=64 solutions at fixed temperature/effort;
# compute accuracy of (a) single sample, (b) majority@k, (c) best-of-k with oracle verifier, (d) best-of-k with a noisy LLM verifier,
# for k in {1,2,4,8,16,32,64}; plot accuracy vs k and vs $ cost; find where the noisy-verifier curve peaks.
```
Extensions: **budget forcing** (length-controlled generation with open models), **adaptive stopping** (stop sampling when agreement ≥ threshold), **difficulty routing** (cheap model first → escalate), **process-reward-guided beam search** on a small math set (GSM8K/MATH subset) with an open model.

## 9. Open questions (state of uncertainty)
Does RL create new reasoning skills or amplify latent ones? How far does test-time scaling extrapolate before returns vanish? How to build verifiers for non-verifiable domains (writing, strategy, science)? How to train models to *use* extra compute productively and **stop** appropriately? Can CoT stay faithful/monitorable under heavy RL? How to evaluate long-horizon agents reliably and cheaply?

## 10. What engineers should do with this
- Treat **thinking effort as a routed resource** (per task difficulty and value).
- Build **verifiers into your systems** (tests, schemas, state checks) — they unlock sample-and-select quality gains and safer automation.
- Always report **budget with accuracy**; compare at equal cost.
- Expect model behavior to change with each release; keep your evals ([[../Phase-7-Evals-Observability-LLMOps/04-Release-Management-and-Experimentation]]).

## Self-check
1. Contrast sequential vs parallel test-time compute with one example each. 2. Why does best-of-N with a noisy verifier eventually degrade? 3. Compute pass@k from n=20 samples, c=6 correct, k=5. 4. What's "verification is easier than generation" and where does it fail? 5. Why are benchmark scores meaningless without inference budget? 6. Why monitor actions not just chain-of-thought?

Next → [[02-Efficient-Models-Quantization-Distillation-Architectures]]
