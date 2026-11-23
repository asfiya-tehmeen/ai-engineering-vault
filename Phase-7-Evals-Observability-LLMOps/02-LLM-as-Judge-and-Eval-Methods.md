---
tags: [phase-7, llm-judge, evals]
---
# 02 — LLM-as-Judge & Advanced Eval Methods

> LLM judges let you evaluate qualities code can't check (faithfulness, tone, completeness, reasoning) at scale. They are **measurement instruments with error** — so validate them like any instrument, and propagate their uncertainty into reported numbers.

## 1. Judge designs
| Design | How | Use |
|---|---|---|
| **Single-answer, binary (pass/fail) + critique** | judge sees (input, output, [context/reference], rubric) → writes a short rationale → label | **recommended default**: actionable, more reliable than scales |
| **Rubric scoring (e.g., 1–5 per criterion)** | anchored descriptions per level | needed for gradations; scores tend to cluster mid-scale unless anchored with examples |
| **Reference-based** | compare to gold answer/facts | correctness checks; judge must not be fooled by phrasing |
| **Reference-free / context-grounded** | check claims against provided sources | faithfulness/groundedness |
| **Pairwise (A vs B)** | choose better or tie | comparing versions/models; randomize order; run both orders |
| **Listwise/ranking** | rank N candidates | reranking/arena style |
| **Multi-judge / panel / jury** | several models/prompts vote | reduces idiosyncratic bias; costs more |
| **Checklist/decomposed** | many yes/no sub-questions → aggregate | more reliable than holistic judgments |
| **Agent-as-judge / execution-augmented** | judge uses tools (run code, query DB, verify facts) | for tasks needing verification |
| **Process/step judges (PRM-style)** | grade intermediate steps | agents/reasoning |

## 2. Writing a good judge prompt
- **One criterion per judge.** Define it precisely with examples of pass and fail (include borderline cases). 
- Provide **all needed context** (question, retrieved docs, policy, conversation) and say what to ignore (e.g., style, length).
- **Reason-then-label:** ask for a brief evidence-based rationale (quotes) *before* the verdict (improves accuracy; also aids debugging). Parse verdict from structured output.
- **Binary decisions with explicit definitions** ("Pass only if every factual claim is supported by CONTEXT; unsupported numbers = Fail").
- **Forbid common shortcuts:** "Do not reward length, formatting, or confident tone."
- Use **few-shot examples** from your labeled set (but hold out evaluation items!).
- **Pin model + version + prompt**; temperature controls where available (note some APIs restrict sampling params — see Phase 4); store judge outputs.
- **Use a different model family** than the system being judged when possible (reduces self-preference bias).
- Prefer a **strong judge** for calibration experiments; then try cheaper judges and measure the agreement loss.

## 3. Known judge biases & mitigations
| Bias | Symptom | Mitigation |
|---|---|---|
| **Position bias** | prefers first/second option | randomize order; evaluate both orders; discard inconsistent pairs |
| **Verbosity/length bias** | prefers longer answers | rubric says ignore length; length-controlled comparison; penalize padding |
| **Self-preference** | favors outputs from its own model family | cross-family judges |
| **Leniency / sycophancy** | passes everything | calibrate thresholds; measure TNR; include known-bad examples |
| **Format/style bias** | likes markdown/lists | normalize format; rubric |
| **Authority/confidence bias** | prefers assertive tone | explicit rule; adversarial examples |
| **Knowledge limits** | can't verify facts it doesn't know | supply sources/tools; don't use judge as fact oracle |
| **Prompt sensitivity** | small wording changes flip results | test prompt variants; ensemble |
| **Judge drift** | scores shift when judge model updates | **pin versions**; recalibrate on any change; keep a frozen calibration set |

## 4. Calibrating a judge against humans (the essential procedure)
1. **Build a human-labeled calibration set** — start with **≥50** examples (more is better; stratify so both passes and failures are well represented — sources commonly suggest ≥50 as a floor). Have 1–2 domain experts label, resolve disagreements, and record **inter-annotator agreement** (Cohen's κ).
2. **Split** into *dev* (prompt tuning/few-shots) and *test* (final measurement).
3. **Run the judge**, compute the **confusion matrix** vs human labels (treat human as truth).
4. **Report per-class metrics**, not just agreement:
   - **TPR (sensitivity)** = P(judge=pass | human=pass) and **TNR (specificity)** = P(judge=fail | human=fail), or precision/recall for the "fail" class you care about. Accuracy alone can mislead on imbalanced data — a judge that passes everything looks 90% accurate if 90% pass.
   - Correlation (Spearman/Pearson) for ordinal scores; rule-of-thumb targets seen in sources: >0.7 correlation with experts, or ≥85–90% agreement before automating, with action if disagreement exceeds ~15% — *treat as heuristics*, set thresholds by decision risk.
5. **Iterate the prompt** on dev errors (read disagreements!), re-measure on test. Stop when error profile is acceptable *for the decision it informs*.
6. **Recalibrate** whenever you change the judge model, prompt, rubric, or task distribution; schedule periodic spot-checks (sources vary — monthly to quarterly; drift can appear within a couple of months). Keep the calibration set frozen and versioned.

## 5. Correcting aggregate estimates for judge error (bias correction)
If you report "pass rate = 82% per judge," that number inherits the judge's errors. With TPR and TNR measured on the calibration set, estimate the **true** pass rate θ from the observed judge pass rate `p_obs`:
```
p_obs = θ·TPR + (1−θ)·(1−TNR)   ⇒   θ̂ = (p_obs + TNR − 1) / (TPR + TNR − 1)       (Rogan–Gladen estimator; valid when TPR+TNR > 1)
```
Then compute **confidence intervals that include both** the sampling error on the eval set **and** the uncertainty in TPR/TNR (bootstrap both stages: resample calibration pairs and eval items). Recent research (2026 preprints on judge bias/uncertainty) emphasizes reporting judge-error-aware intervals. Practical rule: if the judge is weak (TPR+TNR near 1), **don't** use it for absolute claims; at most for coarse A/B direction — or improve it.
```python
import numpy as np
def corrected_rate(p_obs, tpr, tnr): return float(np.clip((p_obs + tnr - 1) / (tpr + tnr - 1), 0, 1))
def bootstrap_ci(judge_eval, calib_h, calib_j, B=2000, rng=np.random.default_rng(0)):
    est = []
    for _ in range(B):
        e = rng.choice(judge_eval, len(judge_eval))                         # resample eval judgments (0/1)
        idx = rng.integers(0, len(calib_h), len(calib_h)); h, j = calib_h[idx], calib_j[idx]
        tpr = ((j == 1) & (h == 1)).sum() / max((h == 1).sum(), 1); tnr = ((j == 0) & (h == 0)).sum() / max((h == 0).sum(), 1)
        if tpr + tnr > 1.05: est.append(corrected_rate(e.mean(), tpr, tnr))
    return np.percentile(est, [2.5, 50, 97.5])
```

## 6. Pairwise comparisons & rating systems
- Randomize A/B order, run both orders, count "consistent wins"; handle ties.
- Aggregate with **Bradley–Terry/Elo** across many comparisons (as in arena-style leaderboards). Report win-rate ± CI; use paired prompts.
- For two prompt versions on your test set: win/tie/loss counts + binomial/paired bootstrap significance.
- Combine with **absolute** checks (a "winner" can still be bad).

## 7. Reference-free faithfulness/groundedness methods
Claim-extraction + entailment (RAGAS-style), NLI models, "judge with evidence quotes," retrieval-grounded checks, **self-consistency**, and **citation verification** (quote must exist in cited source). Known pitfalls: faithful-but-wrong when context is wrong; hidden retrieval misses ([[../Phase-5-RAG-Knowledge-Systems/07-RAG-Evaluation-and-Production]]).

## 8. Synthetic data for evals
- **Structured generation:** define dimensions (persona, intent, difficulty, tone, edge condition, language) → generate combinations → LLM writes realistic inputs → **human review/filter** → label.
- **Perturbation sets:** paraphrase, typos, adversarial rephrasings, injected distractors, negations, long-context padding — to test robustness.
- **Pitfalls:** synthetic questions mirror generator style, skew easy, leak source wording (inflating retrieval metrics), and miss real user quirks. **Keep a real-traffic test set separate** and compare synthetic-vs-real performance gaps.
- Use LLMs to **seed** failure hypotheses, but ground evals in real traces.

## 9. Multi-turn & conversational evals
- Evaluate **conversation-level goals** (resolved? policy followed? clarification asked when needed?) and **turn-level** behaviors.
- **Simulated users** (LLM with scenario/persona/hidden goals) for scalable dialogs; validate the simulator; fix seeds; use multiple trials; beware simulators that can be gamed.
- Test **context carry-over, correction handling, topic switches, interruptions, and long-conversation drift.**

## 10. Safety, robustness & red-team evals
- **Attack success rate (ASR)** for jailbreaks/prompt injection; **over-refusal rate** on benign borderline prompts; policy-violation rates by category; PII leakage tests; bias/fairness slices; tool-misuse tests for agents.
- Datasets/tools: HarmBench, AdvBench-style sets (use responsibly), Garak, PyRIT, promptfoo red-teaming, AgentDojo for agents; plus **domain-specific adversarial cases from your own risk analysis**.
- Pair with **human red-teamers** for creative attacks; track findings → regression tests.

## 11. Contamination & leakage
Public test sets can be in training data; your own test cases can leak via prompts, few-shots, fine-tuning, or retrieval indexes. Mitigate: private held-out sets, canary strings, time-split (post-cutoff) data, rotating/refreshed items, dynamic/live benchmarks, and checking for suspiciously perfect scores.

## 12. Eval tooling landscape (verify current features in docs)
- **Open-source/frameworks:** promptfoo (YAML-driven evals, red-teaming, CI), Inspect (UK AISI; agent & sandbox evals), DeepEval, RAGAS, lm-evaluation-harness, HELM, MLflow GenAI evaluation, OpenAI Evals-style harnesses, pytest + custom.
- **Platforms (tracing + datasets + experiments + judges):** Braintrust, Langfuse (open-source), LangSmith, Arize Phoenix/AX, W&B Weave, Comet Opik, Humanloop-style tools, Galileo, Patronus, Confident AI.
- Choose on: dataset/versioning support, CI integration, judge management & calibration features, trace linking, self-host/data residency, pricing, export/portability. **Own your datasets and graders** in portable formats (JSONL + code) so you can switch tools.

## 13. A calibrated-judge workflow in one page
1. Error analysis → pick failure mode "unsupported claim."  
2. Write binary judge + rubric + examples. 
3. Label 100 outputs by SME (50 dev / 50 test). 
4. Measure TPR/TNR/κ; iterate prompt on dev; freeze; confirm on test.  
5. Use judge in nightly eval; report **bias-corrected pass rate with CI**. 
6. Monitor judge drift via a frozen calibration subset each run. 
7. Re-label disagreements periodically; update dataset v2.

## Self-check
1. Why prefer binary + critique over 1–5 scales? 2. Compute the bias-corrected pass rate: p_obs=0.80, TPR=0.9, TNR=0.8. 3. List four judge biases and a mitigation each. 4. How do you detect judge drift? 5. When is a judge too weak for absolute claims? 6. Design a pairwise eval protocol for two prompt versions.

Next → [[03-Observability-and-Monitoring]]
