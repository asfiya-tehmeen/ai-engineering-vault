---
tags: [phase-9, qa, interview]
---
# 10 — Q&A (Phase 9)

> Answer aloud first, then expand. Frontier answers include uncertainty on purpose — **calibrated "it depends, here's the evidence" beats confident hand-waving** in interviews.

## Reasoning & test-time compute
> [!question]- Q1. What is test-time compute scaling?
> Spending more compute per query — longer chains of thought, multiple samples, search, or refinement — to improve accuracy, giving a second scaling axis alongside pretraining compute, with a per-request cost/latency dial.

> [!question]- Q2. Sequential vs parallel test-time compute?
> Sequential: one long reasoning trace with self-checking/backtracking (trained via RL). Parallel: sample many candidates and select via majority vote or a verifier (best-of-N). Search methods (process-reward-guided tree search) combine both.

> [!question]- Q3. Why does best-of-N with an imperfect verifier eventually degrade?
> The more candidates you generate, the more likely one exploits the verifier's blind spots (reward hacking/over-optimization), so selected answers look good to the verifier but aren't correct; accuracy peaks then falls.

> [!question]- Q4. What does RLVR change in a model?
> It increases the probability of reasoning behaviors that lead to verifiably correct outcomes — deliberation, self-verification, backtracking, decomposition. Whether it creates new capability vs eliciting latent capability is debated; some controlled studies attribute part of the gain to learned termination/format behavior.

> [!question]- Q5. Why are benchmark scores for reasoning models hard to compare?
> Scores depend heavily on inference budget (thinking tokens, samples, tools, retries) and scaffolding; without equal budgets the comparison is meaningless. Report budget with accuracy and compare at equal cost.

> [!question]- Q6. Process vs outcome reward models?
> Outcome models score final answers (sparse, cheap, can reward lucky guesses); process models score intermediate steps (better credit assignment and search guidance but need step labels/noisy synthetic labels).

> [!question]- Q7. "Verification is easier than generation" — implications and limits?
> It makes sample-and-verify strategies work and motivates investing in verifiers/tests. It fails where verification is as hard as generation (open-ended quality, novel science) or when verifiers are imperfect.

> [!question]- Q8. How would you decide thinking effort per request in production?
> Route by task difficulty/value: low effort for classification/routing, higher for planning/debugging; measure accuracy per dollar/second per route; use escalation (cheap first, retry harder on low confidence/failed validation); monitor for overthinking.

> [!question]- Q9. Is chain-of-thought a faithful explanation?
> Not necessarily; it can omit true influences. It's still a useful monitoring surface, so research argues for preserving CoT monitorability. Production safeguards should monitor actions and outcomes, not just reasoning text.

## Efficiency & architectures
> [!question]- Q10. Why is LLM decode memory-bandwidth-bound?
> Each generated token requires reading all weights (and KV cache) for a tiny amount of compute per byte (low arithmetic intensity), far below the GPU's compute-to-bandwidth ridge point; speed ≈ bandwidth ÷ bytes read per token.

> [!question]- Q11. Why does quantizing weights speed up decoding?
> Fewer bytes per parameter means less memory traffic per token; since decode is bandwidth-bound, speed improves nearly proportionally to the byte reduction (subject to dequant overhead and kernel support).

> [!question]- Q12. Why do outliers make quantization hard, and what are mitigations?
> A few channels with huge magnitudes stretch the quantization scale so most values get coarse resolution. Mitigations: per-channel/group scales, mixed-precision outlier handling (LLM.int8), activation-aware scaling (AWQ), migrating outliers (SmoothQuant), rotations/Hadamard transforms.

> [!question]- Q13. QAT vs QAD?
> QAT simulates quantization during training with straight-through estimators using the original loss; QAD trains the quantized student to match the full-precision teacher's output distribution (KL), reported to recover accuracy more stably for models that underwent multi-stage post-training (vendor-reported).

> [!question]- Q14. What are FP8/MXFP4/NVFP4 in one line?
> Low-precision floating-point formats with block/shared scaling that cut memory and increase tensor-core throughput on supporting hardware; vendor-reported accuracy recovery for 4-bit is high on many models but must be verified on your tasks.

> [!question]- Q15. Explain MoE routing and load balancing.
> A router selects top-k experts per token and mixes their outputs by gate weights. Without balancing, a few experts get most tokens (collapse), so an auxiliary loss (or bias adjustment) encourages even utilization; capacity limits and expert parallelism add engineering complexity.

> [!question]- Q16. MoE trade-offs vs dense?
> More capacity per FLOP (total ≫ active params) but all experts must fit in memory, all-to-all communication is costly, routing can be unstable, and batching/serving is more complex.

> [!question]- Q17. Why hybrids of SSM/linear layers with attention?
> Recurrent/linear layers give fixed-size state and linear-time processing but weaker exact recall; a few attention layers restore precise retrieval, giving efficiency with acceptable quality.

> [!question]- Q18. Distillation variants and when each?
> Response-level SFT on teacher outputs (works with black-box teachers, subject to ToS), logit/KL distillation (open teachers, richer signal), on-policy distillation (student samples, teacher scores — reduces mismatch), reasoning-trace distillation. Choose by access, task, and budget; always evaluate on your own tasks.

> [!question]- Q19. How do you evaluate a quantized model properly?
> On your task suite (not just perplexity), including long context, tool/JSON validity, multilingual, safety, on the actual serving engine; report quality with speed/memory.

> [!question]- Q20. What is FlashAttention's key idea?
> IO-aware tiling that keeps blocks in on-chip memory and computes softmax online, avoiding materializing the T×T attention matrix — big memory and speed gains with exact results.

## RL & alignment
> [!question]- Q21. State the policy-gradient estimator.
> ∇J = E[Σ_t ∇log π(a_t|s_t)·(R − b)]; increase the log-probability of actions in proportion to how much better than baseline the outcome was; the baseline reduces variance without bias.

> [!question]- Q22. How does GRPO compute advantages and why drop the critic?
> Sample a group of responses per prompt; advantage = (reward − group mean)/group std, shared across tokens; the group acts as a baseline, avoiding a value network's memory/compute and complexity.

> [!question]- Q23. What happens when all samples in a group get the same reward?
> Advantages are zero, so no gradient signal — hence curriculum/filtering to keep prompts with mixed outcomes.

> [!question]- Q24. Name four reward-hacking patterns and defenses.
> Test special-casing, editing tests/graders, parser/format exploits, verbosity for judge rewards, environment bugs. Defenses: hidden/held-out tests, read-only graders, sandboxing, penalties for suspicious edits, adversarial grader audits, monitoring CoT/actions.

> [!question]- Q25. What is entropy collapse in RL fine-tuning?
> The policy becomes near-deterministic early, shrinking exploration and limiting further improvement; mitigations include entropy bonuses, clipping adjustments, sampling diversity, and filtering strategies.

> [!question]- Q26. What is scalable oversight?
> Methods for supervising systems that outperform their supervisors — e.g., debate, recursive reward modeling, AI-assisted evaluation, weak-to-strong generalization studies.

> [!question]- Q27. "Alignment" vs "control"?
> Alignment aims to make the model's goals/behavior match intent; control assumes a model might be misaligned and designs protocols (monitoring, sandboxing, restricted affordances, human review) so it can't cause harm even then.

> [!question]- Q28. What is emergent misalignment (as studied)?
> Research finding that fine-tuning on a narrow harmful behavior can cause broader misaligned behavior to generalize — a caution about narrow fine-tuning data and side effects. (Interpretation and generality are actively debated.)

> [!question]- Q29. How should application engineers treat alignment claims?
> As one layer, not a guarantee: use defense in depth, least privilege/agency, evals for your risks, monitoring, human oversight, and honest communication of limits.

## Interpretability
> [!question]- Q30. Define superposition and polysemanticity.
> Superposition: representing more features than dimensions via sparse, nearly orthogonal directions. Polysemanticity: individual neurons respond to multiple unrelated features as a consequence.

> [!question]- Q31. Probing vs activation patching?
> Probing shows information is linearly decodable from activations (not that the model uses it). Patching intervenes — swapping activations between runs — to show a component causally affects behavior.

> [!question]- Q32. Describe a sparse autoencoder and its objective.
> An overcomplete autoencoder with a sparsity penalty that decomposes activations into a sparse code over learned feature directions; loss = reconstruction + λ·L1 (or top-k variants). Evaluate via reconstruction, L0, loss recovered, dead features, and causal tests.

> [!question]- Q33. What are attribution graphs and their main caveat?
> Per-prompt graphs of feature-to-feature influence built using a replacement model (e.g., transcoders); the caveat is that the replacement model is an approximation, so discovered mechanisms need validation by perturbation on the original model; analysis is also costly and labor-heavy.

> [!question]- Q34. What are induction heads?
> A two-head circuit (previous-token head + induction head) implementing [A][B] … [A] → [B] in-context copying, linked to in-context learning and appearing as a phase change in training.

> [!question]- Q35. How do you sanity-check an interpretability claim?
> Look for causal interventions, random/shuffled baselines, replication across prompts/seeds/models, blind validation of labels, approximation error of surrogates, and predictive power of the explanation (does steering/ablation do what's predicted).

## Data, continual learning, open problems
> [!question]- Q36. What is model collapse and how do you mitigate it?
> Degradation from recursively training on model-generated data (loss of diversity/tails, error accumulation). Mitigate with real-data mixing, verification/filtering, diversity preservation, and provenance tracking.

> [!question]- Q37. Why is continual learning hard?
> Updating weights with new data tends to overwrite old knowledge (catastrophic forgetting), and safe, efficient online updates at LLM scale remain unsolved; current practice uses retrieval/memory and periodic retraining or adapters.

> [!question]- Q38. How do you evaluate a "breakthrough" claim?
> Separate measured results from lab claims and forecasts; check reproducibility, baselines/budgets, contamination, generalization, and independent replication; prefer trend metrics with clear definitions.

> [!question]- Q39. Name three open problems and a small experiment for each.
> E.g., reliability: measure pass^k vs pass@k on a long-horizon task; verification beyond checkable domains: calibrate a judge and test hacking under optimization; efficient reasoning: adaptive stopping curves for accuracy per token.

> [!question]- Q40. Why does verification cost bottleneck AI for science?
> Progress in RL-style improvement requires reliable feedback; in science, feedback comes from slow/expensive experiments or proofs, limiting iteration speed.

## Research habits
> [!question]- Q41. How do you read a paper efficiently?
> Three passes: survey (claim, figures), understand (method, baselines, metrics, compute), interrogate/reproduce (equations, code, ablations, limitations); keep a read-log with claim, evidence strength, doubts, and actions.

> [!question]- Q42. Red flags in ML papers?
> Cherry-picked examples, missing ablations, gains within noise, weak/untuned baselines, unequal compute budgets, contamination-prone benchmarks, no code, overclaiming.

> [!question]- Q43. How would you reproduce a result with limited compute?
> Choose the qualitative effect at small scale, write hypothesis and metrics first, use small proxies, fixed seeds and multiple runs, compare equal budgets, publish code and negative results.

> [!question]- Q44. How do you stay current without drowning?
> Pick 2–3 trusted curators, prefer primary sources, batch reading, build before reading more, maintain "state-of" docs with dates/confidence, reproduce one paper per quarter, and focus on month-to-month trends.

## Career
> [!question]- Q45. What do employers want from AI engineers (per 2026 career sources)?
> Production-grade Python/SQL and software fundamentals plus LLM systems skills — RAG, agents, evals, monitoring, guardrails, release discipline — with judgment, communication, and evidence of shipped work; prompting alone is a weak differentiator. (Sources are blogs/recruiter content — directional.)

> [!question]- Q46. How do you break in without industry experience?
> Build 2–3 production-style projects with evals and write-ups, contribute to open source, pursue adjacent roles that touch production ML (data/backend/ML-platform/analytics engineering) or internal transfers, network for referrals, and show learning velocity.

> [!question]- Q47. What makes a project hireable?
> Real problem/users, clear metrics and baseline, results with uncertainty, failure analysis, cost/latency numbers, security/privacy notes, tests/CI, a clear README with architecture and limits, and a demo.

> [!question]- Q48. How do you get a referral?
> Build rapport through informational chats and useful contributions, tailor your application, then ask specifically and make it easy (resume + three fit bullets).

> [!question]- Q49. Walk through how you'd evaluate two offers.
> Score learning rate, manager/mentorship, impact/ownership, mission fit, stability, compensation (base/equity/bonus with realistic equity valuation), work-life/travel, location/visa, resume signal, and optionality; talk to engineers on the team; ask about evals/security culture and churn.

> [!question]- Q50. How do you present a project in an interview?
> Problem → users/constraints → design choices and alternatives → evals with numbers → failures and fixes → results → what you'd change; be ready for deep follow-ups on retrieval, judges, security, cost.

> [!question]- Q51. What's your 90-day plan in a new AI role?
> Learn product/customers/codebase, ship a small fix, own a small project end-to-end with an eval/monitoring improvement, document, and deliver a measurable win while building cross-team relationships.

> [!question]- Q52. How do you decide between AI engineer, FDE, and research-engineer paths?
> Match preferences: product building (AI engineer), customer-facing delivery and ambiguity (FDE), experiment-driven scaling work with researchers (research engineer); consider ownership style, travel, depth vs breadth, and team quality.

> [!question]- Q53. How do you handle imposter syndrome in a fast-moving field?
> Track wins, focus on durable fundamentals, learn in public, seek feedback/mentors, accept that nobody knows everything, and measure progress by shipped, evaluated work.

> [!question]- Q54. How do you use AI tools while learning/job hunting responsibly?
> Attempt first, then use AI for explanation/review/mock interviews; verify outputs; follow company policies on AI use in interviews; never misrepresent your skills; periodically practice without AI.

> [!question]- Q55. What durable skills will outlast tool churn?
> Software engineering, measurement/evals, systems thinking, security mindset, communication, learning agility, and domain expertise.

## Synthesis scenarios
> [!question]- Q56. A customer asks whether to adopt a new reasoning model for all workflows. Respond.
> Run your eval suite at equal budgets; compare quality, cost, latency per route; route by difficulty; check migration issues (parameters, tokens, safety), canary rollout, and keep rollback; adopt where measured value justifies cost.

> [!question]- Q57. Leadership asks you to cut inference cost 60% without hurting quality. Plan?
> Profile traffic; caching/batching; route to smaller models with evals; lower effort on easy routes; quantization/distillation or self-hosting where utilization justifies, validating on the served artifact; measure cost per successful task; stage rollout.

> [!question]- Q58. How would you evaluate whether a new open model can replace an API model?
> Build/extend the eval suite for your tasks and slices, include safety/guardrail and long-context/tool tests, serve it with realistic load, measure quality/latency/cost/ops burden, check license/security, shadow then canary.

> [!question]- Q59. What would you work on if given a month of research time?
> Pick a bottleneck with a fast loop — e.g., judge reliability under optimization, agent pass^k improvements via verification, or quantization quality on long-context tool use — write the hypothesis, minimal setup, metrics, and publish results including negatives.

> [!question]- Q60. Where do you think the field is heading, and how do you hedge?
> State uncertainty: test-time compute/RL, agents, efficiency, and evals are clear active areas; continual learning/world models are promising but unproven. Hedge with durable skills, portable eval assets, and continuous reading/experimentation rather than betting on one prediction.

Next → [[11-Resources-and-Practice-Links]]
