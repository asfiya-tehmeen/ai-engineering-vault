---
tags: [phase-9, index, frontier, career]
---
# Phase 9 — Advanced Frontier + Career

> [!abstract] Why this phase exists
> Phases 1–8 made you a **builder and deployer**. This phase has two jobs:
> 1. **Frontier depth** — understand the ideas pushing the field (reasoning/test-time compute, RL for LLMs, efficient architectures and quantization, interpretability, new training paradigms) well enough to read papers, evaluate claims, and reason about where capabilities and constraints are heading.
> 2. **Career launch** — turn everything into a **portfolio, a narrative, a network, and offers**, and build a **learning system** that keeps you current for years.

**Duration:** ongoing. Allow 6–8 weeks for the frontier modules and a 12-week job-search sprint (they can overlap) · **Prereq:** Phases 1–8.

> [!warning] Read this before the frontier notes
> **The frontier moves faster than any text can track, and my sources for it were thin.** Searches for late-2026 research trends returned mostly **blogs, Substacks, forecasts, and preprints** (several only as reference lists). So these notes teach **durable concepts, derivations, and methods** (what test-time compute *is*, how GRPO's advantage works, why quantization error behaves as it does, what an SAE does, how to run a small experiment), and flag current claims as **"reported/contested/unverified."** Treat model names, leaderboard positions, and "breakthrough year" narratives as perishable. The most valuable output of this phase is a **reading-and-experimenting habit** (Note 06), not a snapshot.
>
> The career half draws on **career-blog and recruiter sources** (e.g., claims that hiring managers weight portfolio about as much as education for entry-level roles; reported pay averages) — directional, not authoritative. I found **no layoff or labor-market statistics** in my search, so I make no claims about market conditions beyond what the sources say.

## Topic map
| # | Note | Core ideas |
|---|---|---|
| 01 | [[01-Reasoning-Models-and-Test-Time-Compute]] | CoT, RLVR, test-time scaling methods, verifiers, search, budgets, faithfulness, long-horizon capability |
| 02 | [[02-Efficient-Models-Quantization-Distillation-Architectures]] | quantization (INT8→FP4, QAT/QAD), distillation, MoE, linear attention/SSM hybrids, diffusion LMs, roofline & kernels |
| 03 | [[03-RL-for-LLMs-and-Alignment-Frontier]] | policy gradients → PPO → GRPO, environments, reward hacking, scalable oversight, control, emergent misalignment, safety cases |
| 04 | [[04-Interpretability-and-Understanding-Models]] | probing, logit lens, patching, superposition, SAEs, circuits/attribution graphs, steering |
| 05 | [[05-Data-Continual-Learning-and-Open-Problems]] | data wall & synthetic data, continual learning, memory, world models, multimodal/embodied, open problems & uncertainty |
| 06 | [[06-Reading-Papers-and-Doing-Research]] | three-pass reading, reproduction, ablations, research taste, writing, reading lists, staying current |
| 07 | [[07-Career-Strategy-and-Job-Search]] | market signals, role map, paths by background, portfolio, resume, networking, applications, offers, first 90 days |
| 08 | [[08-Capstone-and-12-Month-Plan]] | integrated capstone, 12-month schedule, learning system, review checkpoints |
| 09 | [[09-Exercises]] | 80+ exercises |
| 10 | [[10-QA-Interview-Questions]] | 60 Q&A (frontier + career) |
| 11 | [[11-Resources-and-Practice-Links]] | papers, courses, tools, communities, career resources |
| 12 | [[12-Projects-and-Final-Checkpoint]] | 6 projects + whole-roadmap mastery assessment |

## Suggested sequencing (12 weeks, parallel tracks)
| Week | Frontier track | Career track |
|---|---|---|
| 1–2 | Note 01 + test-time compute lab | Note 07: audit skills, choose target roles, draft narrative |
| 3–4 | Note 02 + quantization/distillation lab | Portfolio: pick 3 flagship projects, polish READMEs/evals |
| 5–6 | Note 03 + mini-GRPO | GitHub/LinkedIn/resume; publish 1 write-up |
| 7–8 | Note 04 + interp lab | Networking: 10 informational chats; outreach |
| 9–10 | Note 05 + reproduce a paper | Applications (referral-first); mock loops |
| 11–12 | Note 06 habits; capstone | Interviews; offers; decision framework |

## Completion checklist
- [ ] Explain test-time scaling, RLVR, and where each helps/fails; run a best-of-N + verifier experiment with cost–accuracy curves
- [ ] Quantize a model (INT8/INT4) and measure quality vs memory/speed; explain why outliers matter and what QAT/QAD are
- [ ] Explain MoE routing and load balancing; explain why SSM/linear-attention hybrids exist and their trade-offs
- [ ] Derive the policy-gradient estimator and GRPO advantage; run a toy GRPO on a verifiable task and catch reward hacking
- [ ] Explain superposition, train a small sparse autoencoder, and use the logit lens/activation patching
- [ ] Read a paper in three passes, reproduce one key result, and write it up publicly
- [ ] Maintain a living reading/experiment log and a weekly learning cadence
- [ ] Have a polished portfolio (≥3 flagship projects with evals), resume, GitHub, and an outreach/application plan
- [ ] Complete mock loops for AI engineer **and** FDE roles; prepare negotiation notes
- [ ] Capstone + 12-month plan written

Back to [[../00-Roadmap]].
