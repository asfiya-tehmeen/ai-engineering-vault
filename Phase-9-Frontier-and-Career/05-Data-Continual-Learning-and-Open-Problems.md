---
tags: [phase-9, data, continual-learning, open-problems]
---
# 05 — Data, Continual Learning & Open Problems

> **Source note:** 2026 commentary treats **continual learning** as the most-discussed candidate for a new paradigm but **unproven** ("hints but no recognized new law"; prototypes in 2026, wider implementation guessed for ~2027 by one forecast), and calls 2026 a "breakthrough year" for **world models** while another review notes the most prominent world-model demo hasn't become the dominant scaling story. These are **opinion/forecast sources**. This note gives you the concepts and a way to hold uncertainty — not predictions.

## 1. The data frontier
- **Data wall debate:** high-quality human text is finite. Evidence: repeating data up to ~4 epochs is nearly as good as fresh (data-constrained scaling), then returns decay; 2026 work proposes adding repetition/overfitting terms to Chinchilla-style laws; synthetic-token terms are proposed but not settled ([[../Phase-3-Transformers-LLM-Internals/03-Pretraining-and-Scaling]]). Claims about *when* the wall binds are contested.
- **Quality over quantity:** filtering/curation (classifier-based "educational value," dedup, decontamination) often beats scaling raw data; **data mixing/curricula** and **annealing** on high-quality data matter.
- **Mid-training:** injecting reasoning/code/long-context/agentic data before post-training — a major lever in recent pipelines.
- **Synthetic data:** textbooks/QA/code/reasoning traces, rephrasing, **verified** synthetic data (tests/provers/simulators), self-play/self-improvement loops. Risks: **model collapse** (recursive training on own outputs loses diversity/tails), amplified errors/bias, benchmark leakage. Mitigations: verification, diversity injection, mixing with real data, provenance tracking.
- **Beyond text:** video, audio, code execution traces, scientific data, simulation/robotics data, **interaction data from agents** (trajectories with outcomes) — "experience" as the next data source.
- **Data attribution & governance:** licensing, opt-outs, provenance, memorization/extraction, privacy. Legal/regulatory environment is evolving.
- **Open data efforts:** FineWeb-style, Dolma, OLMo, Nemotron-CC — study their pipelines for hands-on learning.

## 2. Continual learning & memory
**Problem:** deployed models are **static** after training; updating them with new knowledge/skills without **catastrophic forgetting** is hard. Humans learn continuously; LLMs rely on retrieval/context as a workaround.
| Approach | Idea | Status/caveats |
|---|---|---|
| **Retrieval/context (RAG, memory files)** | externalize new knowledge | works today; not weight-level learning |
| **Periodic retraining/fine-tuning** | refresh weights on new data | expensive; forgetting; versioning |
| **Regularization/replay** | EWC-style penalties, rehearsal buffers | modest for LLM scale |
| **Parameter-efficient modules** | LoRA adapters per task/tenant, mixtures of adapters | practical for specialization; routing/merging complexity |
| **Model editing** (ROME/MEMIT-style) | change specific facts | brittle, ripple effects, scale limits |
| **Test-time training / fast weights** | update parameters (or memory modules) during inference on the context | research prototypes (e.g., TTT layers, memory-augmented "Titans"-style architectures); efficiency/stability open |
| **Self-improvement loops** | generate → verify → train on successes (STaR/RFT-style) | works in verifiable domains; collapse/overfitting risks |
| **Sleep-time compute / offline consolidation** | agent organizes memory between tasks | emerging idea |
**Engineering takeaway now:** treat **memory as a system design problem** (files/DB/vector stores, compaction, provenance, poisoning defenses — [[../Phase-6-Agents-Tools-MCP/04-Memory-Context-and-State]]) while watching research.

## 3. Long-context vs memory architectures
Long context (1M+ tokens) is available but suffers **context rot**, cost, and weak multi-hop use; **hybrid/linear architectures** (Note 02) reduce cost; **retrieval + compaction** remain the practical answer. Research: learned memory, recurrent state with precise recall, hierarchical summarization, native "scratchpad" memory. Evaluate with long-horizon, multi-needle, and aggregation tasks, not needle-in-haystack alone.

## 4. World models, multimodality & embodiment
- **Native multimodal models** (text+image+audio+video in/out) are increasingly default; challenges: grounding, temporal reasoning, tokenization of continuous signals, evaluation.
- **World models:** learned simulators predicting future observations/states (video/latent dynamics; e.g., interactive-environment generators, JEPA-style predictive architectures). Uses: planning, agent training environments, robotics, game/sim generation. **Open:** fidelity over long horizons, controllability, physical consistency, whether they improve downstream agent performance at scale.
- **Embodied AI / robotics:** **VLA models** (vision-language-action), imitation + RL, sim-to-real, data scarcity, safety/reliability; foundation-model approaches (RT-2/π0-style) show promise but real-world reliability and data collection dominate.
- **AI for science:** protein/molecule design, materials, weather, math/theorem proving, lab automation, literature synthesis; **verifier availability** (experiments/proofs) determines progress; cost of verification is the bottleneck.
- **Computer-use/GUI agents:** reliability and speed improving but brittle; security risks high ([[../Phase-6-Agents-Tools-MCP/02-Tool-Design-and-Environments]]).

## 5. Open problems list (hold with calibrated uncertainty)
1. **Reliability:** consistent behavior (pass^k) on long-horizon tasks; calibrated uncertainty; knowing when to abstain.
2. **Verification beyond checkable domains:** judging quality in writing/strategy/science; robust reward models resistant to hacking.
3. **Efficient reasoning:** more accuracy per token; adaptive compute; stopping rules.
4. **Continual learning/memory:** learn from experience without forgetting or poisoning.
5. **Generalization & data efficiency:** learning from fewer examples; out-of-distribution robustness.
6. **Interpretability at scale:** faithful, cheap, automated understanding of frontier models; using it for monitoring.
7. **Scalable oversight/control:** supervising and containing very capable agents.
8. **Security of agents:** prompt injection is unsolved; formal guarantees (e.g., information-flow control) are early.
9. **Evaluation science:** contamination-resistant, saturation-resistant, reliability-aware benchmarks; judge validity.
10. **Compute/energy/hardware:** efficiency, new accelerators, low-precision training, inference economics.
11. **Societal integration:** labor effects, education, misinformation, governance, concentration of power.
12. **Foundations:** why scaling laws hold; what limits them; mechanisms of in-context learning and emergent behavior.

## 6. How to reason about forecasts and hype
- **Separate** (a) measured results, (b) lab claims, (c) forecasts/opinions. Weigh by **provenance** and **reproducibility**.
- Prefer **trend questions with measurable proxies** (task time horizons, price per capability, benchmark-freshness) over narratives ("breakthrough year").
- **Base rates:** many predicted "paradigm shifts" arrive slower and messier than claimed; many capabilities arrive sooner than skeptics predict — keep wide uncertainty.
- **Hedge your career bets:** invest in **durable skills** (engineering, evals, systems thinking, communication) *and* stay literate about new directions.
- Track a **"what would change my mind"** list for key beliefs.

## 7. Ways to engage with the frontier productively
- **Reproduce** a result at small scale (Note 06).
- **Contribute to open evaluation/tooling** (new benchmarks, grader audits, harness improvements).
- **Write explainers** that force understanding; **maintain a "state of X" doc** with sources and confidence levels.
- **Join reading groups/communities;** discuss with practitioners.
- **Build small research prototypes** (e.g., test-time compute lab, tiny SAE, quantization study) and share the code and honest failures.

## Self-check
1. Summarize the data-wall debate and name two mitigation strategies. 2. List four continual-learning approaches and their limits. 3. Why does verification cost bottleneck AI-for-science? 4. Pick three open problems and propose a small experiment for each. 5. How do you judge a "breakthrough" claim? 6. Which current engineering practices stand in for continual learning?

Next → [[06-Reading-Papers-and-Doing-Research]]
