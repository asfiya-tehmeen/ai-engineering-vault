---
tags: [phase-9, papers, research, learning-system]
---
# 06 — Reading Papers & Doing Research (a system for staying current)

> The field outruns any curriculum. What lasts is a **habit loop**: *find → triage → read → reproduce → write → discuss*. Build it now; it's the difference between a 2026 AI engineer who's still sharp in 2030 and one who isn't.

## 1. How to read a paper (three passes)
**Pass 1 — Survey (5–10 min):** title, abstract, figures/tables, intro's contributions list, conclusion. Decide: *Is this relevant? Credible? Worth another pass?* Write a **one-sentence claim**.
**Pass 2 — Understand (30–60 min):** read the method and experiments; note **what was done, against which baselines, on which data, with what metrics, with what compute**; skip proofs initially; mark unclear parts. Write a **half-page summary in your own words**.
**Pass 3 — Interrogate/reproduce (hours–days):** re-derive key equations, read the code/appendix, try to reproduce a *small* result, run ablations, and list **assumptions, limitations, and ways it could be wrong.**
**Questions to ask every paper**
1. What's the actual claim, and is the evidence **causal** or correlational? 2. What are the **baselines** — strong, tuned, equal compute/data? 3. **Variance**: multiple seeds, CIs? 4. **Contamination/leakage** risks? 5. **Compute/budget** fairness (equal inference tokens?). 6. Does it **generalize** (other models/datasets/scales)? 7. What's **missing** (negative results, failure cases)? 8. Would this change what I **build or measure**?
**Red flags:** cherry-picked examples, no ablations, improvements within noise, comparing against untuned baselines, benchmark-only claims with known contamination, vague methods, no code, huge claims from small tests, citation-padding.

## 2. Finding and triaging papers
- **Sources:** arXiv listings (cs.CL, cs.LG, cs.AI, cs.CR), **Hugging Face Daily Papers**, alphaXiv (discussion), Semantic Scholar/Connected Papers (citation graphs), lab blogs/system cards, conference proceedings (NeurIPS/ICML/ICLR/ACL/EMNLP/COLM), curated newsletters (Latent Space, Interconnects, Ahead of AI, Import AI, The Batch), practitioners on social media/Discords, reading groups.
- **Triage routine (30 min/day or 2 hrs/week):** skim titles → open 5 abstracts → pick 1 for pass 1 → pick 1/week for pass 2 → pick 1/month for pass 3 (reproduce).
- **Prioritize:** papers that (a) change **how you build/evaluate** (eval methods, serving, RAG, agents, security), (b) are **foundational** (seminal), (c) have **code/data**, (d) come from groups with strong empirical hygiene.
- **Keep a "to-read" queue with tags** and a **"read-log"** in the vault: link, date, claim, 3 takeaways, doubts, action.

## 3. Reproduction & experiment hygiene
- **Start tiny:** reproduce the *qualitative* effect on a small model/dataset before scaling. (E.g., induction-head phase change in a 2-layer model; test-time compute curves on a small math set; LoRA vs full FT on one task.)
- **Hygiene:** fixed seeds + multiple runs; config files; experiment tracking (W&B/MLflow); version data/code; log environment; **write the hypothesis and expected result before running**; separate dev/test; report **negative results**.
- **Ablations:** remove one component at a time; compare **equal budgets**; sweep key hyperparameters fairly (including for the baseline).
- **Statistics:** CIs, paired comparisons, multiple-comparison caution ([[../Phase-7-Evals-Observability-LLMOps/01-Eval-Foundations]]).
- **Compute budgeting:** estimate FLOPs/GPU-hours up front; use small proxies; scale only when the trend is clear.
- **Code quality:** readable, tested core functions, README with exact commands; release code so others can reproduce.

## 4. Research taste (what to work on)
- Prefer **questions with fast feedback loops** (experiments in minutes/hours).
- Prefer **measurable claims** and **clear baselines**.
- Look for **gaps between what's claimed and what's measured** (benchmark audits, judge validity, reliability).
- Work on **bottlenecks real systems face** (cost, latency, evals, security, long-horizon reliability).
- Avoid **crowded, compute-heavy races** unless you have an edge; small teams do well on evals, analysis, tooling, data, interpretability, and applied reproducibility.
- **Be wrong fast;** keep a list of hypotheses and kill the weak ones.

## 5. Writing it up
- **Structure:** problem → why it matters → method → setup (data, models, budgets) → results (with CIs) → ablations → limitations → reproducibility (code/data links).
- **Formats:** blog post, short arXiv note, GitHub README, workshop paper, tweet-thread/LinkedIn summary with link. **Public artifacts compound** — they're your portfolio ([[07-Career-Strategy-and-Job-Search]]).
- **Writing habits:** lead with the finding; use figures; avoid overclaiming; include "what didn't work"; credit prior work; invite critique.
- **Peer feedback:** ask 2–3 practitioners to review before posting.

## 6. Reading list (foundations → frontier; arXiv IDs from memory — search titles if needed)
### Foundations (revisit)
Attention Is All You Need (1706.03762) · GPT-3 (2005.14165) · Scaling Laws (2001.08361) · Chinchilla (2203.15556) · LLaMA 3 herd (2407.21783) · DeepSeek-V3 (2412.19437) · InstructGPT (2203.02155) · DPO (2305.18290) · RoPE (2104.09864) · FlashAttention 1–3 (2205.14135, 2307.08691, 2407.08608) · vLLM/PagedAttention (2309.06180) · LoRA (2106.09685) · QLoRA (2305.14314).
### Reasoning & test-time compute
Chain-of-Thought (2201.11903) · Self-Consistency (2203.11171) · Tree of Thoughts (2305.10601) · Let's Verify Step by Step (2305.20050) · Scaling LLM Test-Time Compute Optimally (2408.03314) · s1: Simple test-time scaling (2501.19393) · DeepSeekMath/GRPO (2402.03300) · DeepSeek-R1 (2501.12948) · Are Emergent Abilities a Mirage? (2304.15004).
### RL & alignment
Constitutional AI (2212.08073) · Concrete Problems in AI Safety (1606.06565) · AI Safety via Debate (1805.00899) · Weak-to-Strong Generalization (2312.09390) · Sleeper Agents (2401.05566) · Alignment Faking in LLMs (2412.14093) · Emergent Misalignment (2502.17424) · AI Control (2312.06942) · Towards Understanding Sycophancy (2310.13548) · "Chain of Thought Monitorability" (search title).
### Efficiency & architectures
LLM.int8() (2208.07339) · GPTQ (2210.17323) · AWQ (2306.00978) · SmoothQuant (2211.10438) · FP8 Formats for DL (2209.05433) · NVFP4 QAD (2601.20088) · Switch Transformer (2101.03961) · Mixtral (2401.04088) · DeepSeekMoE (2401.06066) · Mamba (2312.00752) · Mamba-2 (2405.21060) · Gated DeltaNet (2412.06464) · TTT layers (2407.04620) · Titans (2501.00663) · LLaDA diffusion LM (2502.09992) · GKD on-policy distillation (2306.13649).
### Interpretability
A Mathematical Framework for Transformer Circuits (transformer-circuits.pub) · In-context Learning and Induction Heads · Toy Models of Superposition (2209.10652) · Towards Monosemanticity · Scaling Monosemanticity · Sparse Autoencoders Find Highly Interpretable Features (2309.08600) · Locating and Editing Factual Associations (ROME, 2202.05262) · Representation Engineering (2310.01405) · Refusal in LMs Is Mediated by a Single Direction (2406.11717) · Circuit Tracing / On the Biology of a Large Language Model (transformer-circuits.pub 2025).
### Agents, RAG, evals, security
ReAct (2210.03629) · Toolformer (2302.04761) · SWE-bench (2310.06770) · τ-bench (2406.12045) · Lost in the Middle (2307.03172) · RAG (2005.11401) · RAGAS (2309.15217) · Indirect Prompt Injection (2302.12173) · AgentDojo (2406.13352) · Judging LLM-as-a-Judge (2306.05685) · Adding Error Bars to Evals (2411.00640).
### Data & scaling
Scaling Data-Constrained LMs (2305.16264) · Textbooks Are All You Need (2306.11644) · The Curse of Recursion / model collapse (2305.17493) · FineWeb (search title) · Continual learning of LLMs survey (2404.16789).
**How to use the list:** one paper per week; alternate *foundational* and *recent*; write a read-log entry for each; reproduce one per quarter.

## 7. Contributing without a PhD
- **Open-source:** evals/harnesses (Inspect, lm-eval-harness), serving (vLLM/SGLang issues), RAG/agent frameworks, interpretability tools (TransformerLens/SAELens), docs/cookbooks. Start with "good first issues," then fix real bugs you hit.
- **Benchmarks/audits:** find and document flaws in an agent/eval benchmark (leakage, gameable graders); propose fixes.
- **Replications & negative results:** valuable and underprovided.
- **Fellowships/programs (availability varies):** research fellowships/residencies, open-source programs (e.g., Google Summer of Code), safety fellowships (e.g., MATS-style), hackathons.
- **Collaborate:** find a peer/mentor; co-author small studies.

## 8. The learning system in your vault (implementation)
```
00-Learning-OS/
  reading-log.md          # one entry per paper (template below)
  experiment-log/         # one note per experiment: hypothesis → setup → result → decision
  state-of/               # living docs: "state of test-time compute", "state of agents security" with sources + confidence + date
  flashcards.md           # Q::A cards (Spaced Repetition plugin) from Q&A notes
  weekly-review.md        # 5 lines: learned, built, confused, next, energy
  questions.md            # open questions & hypotheses
```
**Read-log template:** `Title | link | date | Claim (1 line) | Method | Evidence strength | 3 takeaways | Doubts | Action (reproduce? apply? ignore?)`
**Weekly cadence:** 2 hrs reading, 4–6 hrs building/experiments, 1 hr writing, 30 min review (flashcards + log), 1 community interaction (reading group/chat/PR).
**Quarterly:** reproduce one paper; ship one public write-up; refresh your "state of" docs; prune your queue; update goals.

## 9. Staying sane about information overload
Pick **2–3 trusted curators** and ignore the rest; batch consumption; prefer **primary sources** over summaries; separate **learning** from **entertainment/FOMO**; build before you read more; keep a **"not now"** list; remember most week-to-week noise doesn't matter — **month-to-month trends** do.

## Self-check
1. Explain the three-pass method and what you produce at each. 2. List eight critical questions for any paper. 3. Design a tiny reproduction of a result from Note 01–04 with hypothesis and metrics. 4. Draft a read-log entry for a paper you know. 5. Outline your weekly learning cadence. 6. Name three ways to contribute without a PhD and the first concrete step for each.

Next → [[07-Career-Strategy-and-Job-Search]]
